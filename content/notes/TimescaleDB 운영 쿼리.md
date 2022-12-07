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

