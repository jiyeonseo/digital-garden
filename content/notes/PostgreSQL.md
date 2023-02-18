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

## pg_user
### select all user
```sql
SELECT 
	usename AS role_name, 
	CASE 
		WHEN usesuper AND usecreatedb 
			THEN CAST('superuser, create database' AS pg_catalog.text) 
		WHEN usesuper 
			THEN CAST('superuser' AS pg_catalog.text) 
		WHEN usecreatedb 
			THEN CAST('create database' AS pg_catalog.text) 
		ELSE 
			CAST('' AS pg_catalog.text) 
	END role_attributes 	
FROM pg_catalog.pg_user 
ORDER BY role_name desc;
```

### create user
```sql
CREATE USER <name>;
CREATE USER <name> WITH PASSWORD '<password>';
ALTER USER <name> WITH PASSWORD '<password>';
```


### grant privileges
```sql
GRANT ALL PRIVILEGES ON DATABASE <database> TO <name>;
SELECT * from information_schema.table_privileges WHERE grantee = '<name>' LIMIT 5;
```

## References
- [PostgreSQL: 베큠(VACUUM)을 실행해야되는 이유 그리고 성능 향상](https://blog.gaerae.com/2015/09/postgresql-vacuum-fsm.html)
- https://tableplus.com/blog/2018/04/postgresql-how-to-grant-access-to-users.html