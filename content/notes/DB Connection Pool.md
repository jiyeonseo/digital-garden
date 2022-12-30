---
title: "DB Connection Pool이 필요한 이유"
tags:
- database
---
## DB Connection
어플리케이션이 DB server와의 통신을 위한 연결로 SQL 문을 보내고 그 결과값을 받기 위해 사용한다. 데이터베이스 서버도 앱서버와 같이 특정 포트 (예를 들어, MySQL 데이터베이스 서버는 기본 `3306`)로 떠 있으며, 이 서버와 백엔드 서버가 **TCP-IP protocol**로 Connection을 맺는다. 이때, 필요에 따라 user name, password와 같은 credentials 가 필요할 수도 있다.

DB Connection을 맺기 위해서는 DB Host, Port, database name, driver, user name, password 등이 필요하다. 

```
db_url      = jdbc:mysql://HOST/DATABASE  
db_driver   = com.mysql.jdbc.Driver  
db_username = USERNAME  
db_password = PASSWORD
```

## DB Connection 생애주기
![](https://vladmihalcea.com/wp-content/uploads/2014/04/connectionlifecycle.gif)
(ref : https://vladmihalcea.com/wp-content/uploads/2014/04/connectionlifecycle.gif)
1. DB로 Connection을 처음 연결 시도 한다.
2. Connection을 실제로 열기 전, user credential 이 유효한지 인증 검사를 한다. 
3. 검사를 통과하면 TCP socket을 연다.
4. SQL 문이 들어오면 이에 대한 결과 데이터를 보내준다. 
5. DB Connection을 닫는다.
6. TCP socket이 닫힌다. 

위에서 보다시피 DB Connection을 맺는 것은 매우 비싸고 시간이 많이 드는 작업이다. 그렇기 때문에 필요할 때마다 connection을 맺는 것은 어플리케이션 자체 속도를 늦춰서 서비스 품질에 악영향을 끼칠 수 있다. 또 각각의 요청마다 connection을 맺게되면 동시에 너무 많은 connection이 만들어지게 되며 DB server의 CPU와 memory를 너무 많이 사용하여 resource 부족 문제를 야기할 수 있다. 

이 문제를 풀기 위하여 Connection Pool을 사용한다. 미리 Connection을 맺어두고 어플리케이션에서는 이미 만들어진 Connection을 재사용(reuse)한다. 

![](https://user-images.githubusercontent.com/2231510/209923693-a45f6129-3053-4089-a626-ef77143b6032.png)
(ref:https://raw.githubusercontent.com/practicalli/graphic-design/live/practicalli-clojure-webapps-database-postgres-no-connection-pool.png)


## References
- [Why do we need a Database Connection Pool? - every programmer must know](https://dineshchandgr.medium.com/why-do-we-need-a-database-connection-pool-every-programmer-must-know-9f90e7c8e5af)