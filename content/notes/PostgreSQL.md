---
title: "PostgreSQL"
tags:
- database
---

## Vacuum
- DB 디스크 조각 모음
- 데이터가 `update`/`delete` 
- 
### vacuum 시간 확인하기
```sql
SELECT relname, last_vacuum, last_autovacuum FROM pg_stat_user_tables;
```


## References
- [PostgreSQL: 베큠(VACUUM)을 실행해야되는 이유 그리고 성능 향상](https://blog.gaerae.com/2015/09/postgresql-vacuum-fsm.html)