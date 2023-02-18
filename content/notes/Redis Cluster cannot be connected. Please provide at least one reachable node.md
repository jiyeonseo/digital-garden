---
title: "[redis-py Cluster] Redis Cluster cannot be connected. Please provide at least one reachable node"
tags:
- redis
---

[[notes/memorydb]] 를 node 2개(master + replica)로, client는 [redis-py](https://github.com/redis/redis-py) cluster로 연결하고 사용하고 있었다. Memory DB는 Redis Cluster로 구성되어있기 때문에 하나의 node가 내려가더라도 대기중인 replica가 올라가기 때문에 내구성이 뛰어나고, 모든 노드에 똑같이 복사가 되기 때문에 데이터의 유실도 없다.  

redis-py 는 아래와 같이 client 설정을 해주었다. 
```py
from redis.asyncio.cluster import RedisCluster as AsyncRedisCluster

...

REDIS_CONNECTION_PARAMS = {
	"password": app_settings.redis_password,
	"username": app_settings.redis_user,
	"host": app_settings.redis_endpoint,
	"port": app_settings.redis_port,
	"encoding": "utf-8",
	"ssl": True,
	"decode_responses": True,
	"require_full_coverage": False,
}

self._redis_cluster = AsyncRedisCluster(**REDIS_CONNECTION_PARAMS)
...
```

그러던 어느날, Client쪽에서 다음과 같은 에러가 떨어졌다. 
```
Redis Cluster cannot be connected. Please provide at least one reachable node: None
```
즉, 붙을 수 있는 node가 없다는 에러다. 확인한 결과 분명 Momory DB는 떠 있는 상태였다. 다만 이벤트를 확인해 보았을 때, 어떠한 이유로 master node가 내려간 것으로 보인다. 

![unnamed__3_](https://user-images.githubusercontent.com/2231510/218296467-ab03ff37-b8c3-4108-a684-27c882de628e.png)
- master node 였건 `*-0001-002` 가 어떠한 이유로 Failover되어 replica node 였던 `*-0001-001` 가 master로 승격되었고,
- 바로 이어서 다시 `*-0001-001`에 Failover 되며 다시 `*-0001-002`  가 master로 승격되었다. 

이후, 위 에러가 발생하였다. 분명 node가 내려가더라도 서비스 장애 없이 잘 되어야 할 것 같은데 node를 찾지 못했다. 

## MemoryDB Failover 테스트하기 

우선 현상을 재현해보기 위해 Memory DB의 master node를 일부러 내리는 테스트를 해보았다. 두가지 방법으로 master node를 내려볼 수 있다. 

### 1) AWS Console
![](https://user-images.githubusercontent.com/2231510/218296636-63f167a0-a7d6-4024-b4b4-ae147f0407d7.png)
- AWS MomoryDB Console -> 테스트할 클러스터 선택 > 상단 "프라이머리 장애 조치"  
- 제대로 실행 되면 왼쪽 "Events" 탭에 장애 조치 시작인 "FailoverSharedAPI Called" 와 완료 "Recovery completed" 되는 이벤트들을 확인 할 수 있다. 

### 2) AWS CLI
```
# Linux, macOS, Unix
aws memorydb failover-shard \  
   --cluster-name my-cluster \  
   --shard-name 0001

# Windows
aws memorydb failover-shard ^  
   --cluster-name my-cluster ^  
   --shard-name 0001
```
위와 같이 요청하면, 다음과 같은 응답값이 온다. 
```

{  
     "Events": [  
        {  
             "SourceName": "my-cluster",  
             "SourceType": "cluster",  
             "Message": "Failover to replica node my-cluster-0001-002 completed",  
             "Date": "2021-08-22T12:39:37.568000-07:00"  
         },  
         {  
             "SourceName": "my-cluster",  
             "SourceType": "cluster",  
             "Message": "Starting failover for shard 0001",  
             "Date": "2021-08-22T12:39:10.173000-07:00"  
         }  
    ]  
 }
```

### 유의 사항 
- 클러스터당 24시간에 최대 5번 사용 가능
- 다른 클러스터의 샤드에서 작업을 동시에 호출 할 수 있다. 
- 연속 호출되는 경우 첫 번째 노드 교체가 완료되어야 그 다음 요청 호출 가능하다. 
- 이 기능은 어플리케이션 동작을 테스트 하기 위한 것이며 운영을 위한 기능이 아니다. 따라서 대규모 운영 환경에서는 AWS 자체적으로 해당 요청을 차단 할 수도 있다. 

## redis-py 확인하기

위 테스트를 몇번 해 본 결과, 첫번째 장애 이후에는 문제없이 잘 동작한다. 하지만 두번째 장애가 난 후에는 계속하여 노드를 찾지 못한다. 

### 1)  `reinitialize_steps` 설정 
Redis Client 설정값 중 `reinitialize_steps` 라는 녀석이 있다.  ([redis-py/redis/cluster.py](https://github.com/redis/redis-py/blob/428d60940f386d3680a413aa327889308f82c5de/redis/cluster.py#L506-L515))

> reinitialize_steps (int, default: 5) –
> Specifies the number of MOVED errors that need to occur before reinitializing the whole cluster topology. If a MOVED error occurs and the cluster does not need to be reinitialized on this current error handling, only the MOVED slot will be patched with the redirected node. To reinitialize the cluster on every MOVED error, set reinitialize_steps to 1. To avoid reinitializing the cluster on moved errors, set reinitialize_steps to 0.

즉, `MOVED error`가 난 slot에 대해서 redirected node로 패치를 해준다는 것인데. 
`MOVEDError`([redis-py/redis/exceptions.py](https://github.com/redis/redis-py/blob/master/redis/exceptions.py#L173-L180))란 cluster에서 해당 키가 없는 노드에다가 요청하면 Cluster 에서 올바른 노드를 알려주는 Error다. 
```py 
class MovedError(AskError):
    """
    Error indicated MOVED error received from cluster.
    A request sent to a node that doesn't serve this key will be replayed with
    a MOVED error that points to the correct node.
    """

    pass
```

Redis client를 통해 command를 실행하였을 때 MOVED error가 발생하게 되면 내부에서 `_should_reinitialized` 함수가 불리우게 된다. ([redis-py//redis/cluster.py](https://github.com/redis/redis-py/blob/master/redis/cluster.py#L1134-L1147)) 
```py
...
	except MovedError as e:
		# First, we will try to patch the slots/nodes cache with the
		# redirected node output and try again. If MovedError exceeds
		# 'reinitialize_steps' number of times, we will force
		# reinitializing the tables, and then try again.
		# 'reinitialize_steps' counter will increase faster when
		# the same client object is shared between multiple threads. To
		# reduce the frequency you can set this variable in the
		# RedisCluster constructor.
		self.reinitialize_counter += 1
		if self._should_reinitialized():
			self.nodes_manager.initialize()
			# Reset the counter
			self.reinitialize_counter = 0
		else:
			self.nodes_manager.update_moved_exception(e)

		moved = True
...
```

이 함수 내에서 Redis Client 설정시 넘겨주었던 `reinitialized_steps` 가 사용된다. 
```py
def _should_reinitialized(self):
	# To reinitialize the cluster on every MOVED error,
	# set reinitialize_steps to 1.
	# To avoid reinitializing the cluster on moved errors, set
	# reinitialize_steps to 0.

	if self.reinitialize_steps == 0:
		return False
	else:
		return self.reinitialize_counter % self.reinitialize_steps == 0
```
따라서, 넘겨준 n번의 MOVED error가 발생하였을 때, 그 때 reinitialize 해주는 로직이다. MOVED가 일어날 때 마다 reinitialize 해주려면 `1`로, 일어나지 않게 하려면 `0`으로 세팅하면 된다. (기본 값은 `5`이다)

위 상황을 미뤄 보았을 때, 변경된 node를 알아채지 못하여 reachable node를 못찾은 건가 싶어 `1`로 설정하고 다시 테스트를 해보았다. 하지만 여전히 해결되지 못했다. 

### 2) `NodeManager` 이슈

정확하게 저 에러 메세지가 일어나는 부분부터 다시 확인을 하였다. 
에러가 난 부분은 아래 코드이다. ([redis-py/redis/cluster.py](https://github.com/redis/redis-py/blob/master/redis/cluster.py#L1551-L1555))
```py
class NodesManager:
...
	def initialize(self):
	...
	startup_nodes_reachable = False
	
	for startup_node in self.startup_nodes.values():
		startup_nodes_reachable = True
	
	...
	if not startup_nodes_reachable:
		raise RedisClusterException(
			f"Redis Cluster cannot be connected. Please provide at least "
			f"one reachable node: {str(exception)}"
		) from exception
```

`NodeManager` 에서 initialize 시에 reachable한 node가 없는 경우 생기는 이슈다. 여기서 `startup_nodes_reachable`는 `NodeManager` 생성시 넘어온 startup_nodes를 루프를 돌며 설정이 되는데 이때, 모든 node가 reachable 하지 못했던 것이였다. 

여기서 사용한 `self.startup_nodes`는 `RedisCluster` 처음 생성될 때 url 혹은 host, port 정보를 통해 Cluster에 등록된 node들을 한번에 가져와 등록을 해준다. ([redis-py/redis/cluster.py](https://github.com/redis/redis-py/blob/master/redis/cluster.py#L534-L554)) 

```py
class RedisCluster(AbstractRedisCluster, RedisClusterCommands):
...
def __init__(
	self,
	host: Optional[str] = None,
	port: int = 6379,
	startup_nodes: Optional[List["ClusterNode"]] = None,
	cluster_error_retry_attempts: int = 3,
	retry: Optional["Retry"] = None,
	require_full_coverage: bool = False,
	reinitialize_steps: int = 5,
	read_from_replicas: bool = False,
	dynamic_startup_nodes: bool = True,
	url: Optional[str] = None,
	**kwargs,
	):
	...
	if url is not None:
		url_options = parse_url(url)
		...
		kwargs.update(url_options)
		host = kwargs.get("host")
		port = kwargs.get("port", port)
		startup_nodes.append(ClusterNode(host, port))
	elif host is not None and port is not None:
		startup_nodes.append(ClusterNode(host, port))
	...
	self.nodes_manager = NodesManager(
		startup_nodes=startup_nodes, # 여기! 
		from_url=from_url,
		require_full_coverage=require_full_coverage,
		dynamic_startup_nodes=dynamic_startup_nodes,
		**kwargs,
	)
```

이 과정은 RedisClient가 처음 init 될때 생성한  `startup_nodes` 로 `__init__` 이후에는 변하지 않는다. 즉, 처음 RedisClient 객체가 생성할 때 가지고 온 node로 계속하여 initialize를 하는 것이다. `NodeManager`에서는 node의 IP addr를 보고 있기 때문에, node가 내려갔다 다시 뜨면서 IP가 변경되면서 `NodeManager`에서는 해당 node를 찾을 수 없다.

내 상황의 경우 cluster에 유효한 node가 처음 뜰 때 `startup_node` 로 등록되었고 2번 failover를 하게되면 더 이상 이 Cluster Client가 알 수 있는 node를 모두 사용했기 때문에 더 이상 reachable node가 없는 것이였다.찾아보니 [이미 2022년 11월에 redis-py issue](https://github.com/redis/redis-py/issues/2472)로 올라와있는 상태였고, 2023년 2월 현재 아직 fix가 되지 않은 상태이다. 이상적인 방법은 Client가 reinitialize 될때, 
라이브러리를 업데이트를 기다리고 있을 수만은 없어 exception 발생시 다시 Client 자체를 새로 생성하여 `__init__`을 다시 하여 node를 찾을 수 있게 임시방편을 추가했다. 