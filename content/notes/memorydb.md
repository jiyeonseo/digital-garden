---
title: "MemoryDB"
tags:
- aws
- redis
---
- [Redis Cluster](https://redis.io/docs/management/scaling/)로 다중AZ 트랜잭션으로 빠르게 읽으면서도 내구성있는 DB. 
- 따로 RDB + Cache 구성 필요 없이, Redis 구성 하나만으로 Consistency와 속도의 장점을 모두 가질 수 있다.

## 특징 
- 복제본 노드에 대한 일관성 보장
- milisecond 단위 write와 microsecond 단위의 read
- Redis client 그대로 사용 가능
### Cluster
- 노드의 모음
	- primary node : 읽기/쓰기
        - readonly : 읽기
- 데이터 세트는 샤드로 분할되어 저장됨
### Node
- Amazon EC2 instance
### Shard
- primary

## Terraform 으로 Cluster 만들기

```sql
resource "aws_memorydb_cluster" "example" {
  acl_name                 = "open-access"
  name                     = "my-cluster"
  node_type                = "db.t4g.small"
  num_shards               = 2
  security_group_ids       = [aws_security_group.example.id]
  snapshot_retention_limit = 7
  subnet_group_name        = aws_memorydb_subnet_group.example.id
}
```

-   [(Terraform) ****Resource: aws_memorydb_cluster****](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/memorydb_cluster)

## References
- [Amazon MemoryDB for Redis](https://docs.aws.amazon.com/memorydb/latest/devguide/what-is-memorydb-for-redis.html)
