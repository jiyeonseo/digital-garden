---
title: "TimescaleDB 운영 쿼리"
tags:
- TimescaleDB
---
---
## Timescale DB Version 확인
```sql
SELECT extversion FROM pg_extension where extname = 'timescaledb';
```

## Hypertable 만들기
```sql
SELECT create_hypertable('public.cheese_table_name', 'time');
```

## Hypertable interval 설정하기
```sql
SELECT set_chunk_time_interval('public.cheese_table_name', INTERVAL '24 hours');
SELECT set_chunk_time_interval('public.cheese_table_name', INTERVAL '3 days');
```
- doc : [Change hypertable chunk intervals](https://docs.timescale.com/timescaledb/latest/how-to-guides/hypertables/change-chunk-intervals/#change-the-chunk-interval-length-on-an-existing-hypertable)

## 테이블 사이즈 확인하기 
```sql
SELECT table_name, pg_size_pretty(pg_relation_size(quote_ident(table_name))) , pg_relation_size(quote_ident(table_name)) 
FROM information_schema.tables 
WHERE table_schema = 'public' 
ORDER BY 3 
```

## Internal Table 까지 모두 사이즈 확인하기 (hypertable, compressed 모두 포함 )
```sql


SELECT table_schema, table_name, pg_size_pretty(pg_relation_size('"'||table_schema||'"."'||table_name||'"')) , pg_relation_size('"'||table_schema||'"."'||table_name||'"') 
FROM information_schema.tables 
ORDER BY 4
```

## Compression 하기
```sql

-- 5일보다 오래된 chunk 찾기 
SELECT show_chunks('cheese_table_name', older_than => INTERVAL '5 days');

-- 특정 chunk compression 하기 
SELECT compress_chunk( '<chunk_name>');
SELECT compress_chunk('_timescaledb_internal._hyper_5_53187_chunk');

-- compression 결과 확인 
SELECT * FROM chunk_compression_stats('example');

-- 기간 내 compression 안된게 있는 경우 수동 compression 
SELECT compress_chunk(i, if_not_compressed => true)
    FROM show_chunks(
        'example',
        now()::timestamp - INTERVAL '1 week',
        now()::timestamp - INTERVAL '3 weeks'
    ) i;

```

doc : [Manual compression](https://docs.timescale.com/timescaledb/latest/how-to-guides/compression/manually-compress-chunks/)

## Decompression 하기
```sql
SELECT decompress_chunk('_timescaledb_internal.<chunk_name>');
```

doc : [Decompression](https://docs.timescale.com/timescaledb/latest/how-to-guides/compression/decompress-chunks/)[](https://docs.timescale.com/timescaledb/latest/how-to-guides/compression/decompress-chunks/)

## Compression된 Hypertable에 데이터 다시 넣기 (Backfill)
```sql
-- chunk 확인하기 
SELECT show_chunks('public.cheese_table_name'); 

-- policy 먼저 확인하기 
SELECT j.job_id
    FROM timescaledb_information.jobs j
    WHERE j.proc_name = 'policy_compression'
        AND j.hypertable_name = 'public.cheese_table_name'; -- 1006

-- policy 멈춰주기 
SELECT alter_job(<job_id>, scheduled => false);
SELECT alter_job(1006, scheduled => false);

-- chunk decompression 하기 
SELECT decompress_chunk('_timescaledb_internal.<chunk_name>');

-- policy 다시 활성화 시켜주기
SELECT alter_job(<job_id>, scheduled => true);
SELECT alter_job(1006, scheduled => true);
```

## 등록된 모든 job 확인하기 
- refresh continous aggregate policy
- compression policy 
```sql
SELECT * FROM timescaledb_information.jobs;
```

## Replication lag 체크하기 
```sql
SELECT 
  CASE 
    WHEN pg_last_wal_receive_lsn() = pg_last_wal_replay_lsn() THEN 0
    ELSE EXTRACT (EPOCH FROM now() - pg_last_xact_replay_timestamp()) 
  END AS log_delay;
```

## Job 
- retention, continuous aggregate 등 주기적으로 작업되는 policy들은 모두 job으로 등록되어있다. 
- doc : [Actions and automation](https://docs.timescale.com/api/latest/actions/)

### 등록된 job 확인
```sql
SELECT *
FROM timescaledb_information.jobs j
```

```markdown
|job_id|application_name|schedule_interval|max_runtime|max_retries|retry_period|proc_schema|proc_name|owner|scheduled|config|next_start|hypertable_schema|hypertable_name|check_schema|check_name|

|------|----------------|-----------------|-----------|-----------|------------|-----------|---------|-----|---------|------|----------|-----------------|---------------|------------|----------|

|1|Telemetry Reporter [1]|24:00:00|00:01:40|-1|01:00:00|_timescaledb_internal|policy_telemetry|postgres|true||2022-12-18 08:02:08.220 +0900|||||

|1001|Refresh Continuous Aggregate Policy [1001]|01:00:00|00:00:00|-1|01:00:00|_timescaledb_internal|policy_refresh_continuous_aggregate|nftbankci|true|{"end_offset": "01:00:00", "start_offset": "3 days", "mat_hypertable_id": 7}|2022-12-17 12:48:56.433 +0900|_timescaledb_internal|_materialized_hypertable_7|_timescaledb_internal|policy_refresh_continuous_aggregate_check|

|1003|Compression Policy [1003]|35 days|00:00:00|-1|01:00:00|_timescaledb_internal|policy_compression|nftbankci|true|{"hypertable_id": 7, "compress_after": "5 days"}|2023-01-05 20:39:47.486 +0900|_timescaledb_internal|_materialized_hypertable_7|_timescaledb_internal|policy_compression_check|

|1011|Retention Policy [1011]|1 day|00:05:00|-1|00:05:00|_timescaledb_internal|policy_retention|nftbankci|true|{"drop_after": "90 days", "hypertable_id": 20}|2022-12-17 16:55:15.014 +0900|_timescaledb_internal|_materialized_hypertable_20|_timescaledb_internal|policy_retention_check|

```

- `Refresh Continuous Aggregate Policy`
	- continuous aggregate 내가 지정한 테이블 이름으로 보이지 않는다. 내부적으로는 internal table로 되어있다. 
	```sql
	   select * from _timescaledb_internal._materialized_hypertable_20
	```
	
- `Compression Policy`
	- 일정 기간이 지난 데이터는 압축한다. 
	- read만 가능 update, delete를 위해서는 uncompression을 따로 해주어야 한다.
	- doc : [Compression](https://docs.timescale.com/api/latest/compression/)
- `Retention Policy`
	- 일정 기간이 지난 후에는 자동 삭제한다. 
	- doc : [Data Retention](https://docs.timescale.com/api/latest/data-retention/)


### job 삭제
```sql
SELECT delete_job(<job id>);
SELECT delete_job(1000);
```