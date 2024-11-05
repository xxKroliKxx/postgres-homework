
# №9 Слабоструктурированные данные и асинхронная обработка

```
create table if not exists json_data
(
    id   serial primary key,
    data jsonb
);

insert into json_data (data)
select jsonb_build_object(
               'name', md5(random()::text),
               'email', lower(substring(md5(random()::text), 1, 8)) || '@example.com',
               'phone', '+7(' || (100 + trunc(random() * 900))::int || ')' ||
                        (100 + trunc(random() * 900))::int || '-' ||
                        (10 + trunc(random() * 90))::int || '-' ||
                        (10 + trunc(random() * 90))::int,
               'value', random() * 1000
       )
from generate_series(1, 1000000);

create index if not exists idx_json_data on json_data using gin (data);
create index if not exists idx_json_data_email ON json_data ((data ->> 'email'));

select pg_size_pretty(pg_total_relation_size('json_data'))                                 AS total_size,      -- 485 MB
       pg_size_pretty(pg_relation_size('json_data'))                                       AS main_table_size, -- 174 MB
       pg_size_pretty(pg_total_relation_size('json_data') - pg_relation_size('json_data')) AS toast_size; -- 311 MB

UPDATE json_data
SET data = jsonb_set(data, '{value}', ((data ->> 'value')::numeric + 100)::text::jsonb);

select pg_size_pretty(pg_total_relation_size('json_data'))                                 AS total_size,      -- 636 MB
       pg_size_pretty(pg_relation_size('json_data'))                                       AS main_table_size, -- 269 MB
       pg_size_pretty(pg_total_relation_size('json_data') - pg_relation_size('json_data')) AS toast_size; -- 367 MB

vacuum full json_data;

select pg_size_pretty(pg_total_relation_size('json_data'))                                 AS total_size,      -- 484 MB
       pg_size_pretty(pg_relation_size('json_data'))                                       AS main_table_size, -- 174 MB
       pg_size_pretty(pg_total_relation_size('json_data') - pg_relation_size('json_data')) AS toast_size; -- 311 MB

explain analyse
SELECT *
FROM json_data
WHERE data @> '{"email": "947703b7@example.com"}';

-- Bitmap Heap Scan on json_data  (cost=47.62..428.75 rows=100 width=147) (actual time=0.178..0.179 rows=1 loops=1)
-- "  Recheck Cond: (data @> '{""email"": ""947703b7@example.com""}'::jsonb)"
--   Heap Blocks: exact=1
--   ->  Bitmap Index Scan on idx_json_data  (cost=0.00..47.60 rows=100 width=0) (actual time=0.169..0.169 rows=1 loops=1)
-- "        Index Cond: (data @> '{""email"": ""947703b7@example.com""}'::jsonb)"


explain analyse
select *
from json_data
where data ->> 'email' = '947703b7@example.com';

-- Bitmap Heap Scan on json_data  (cost=139.18..12129.40 rows=5000 width=147) (actual time=0.031..0.032 rows=1 loops=1)
--   Recheck Cond: ((data ->> 'email'::text) = '947703b7@example.com'::text)
--   Heap Blocks: exact=1
--   ->  Bitmap Index Scan on idx_json_data_email  (cost=0.00..137.93 rows=5000 width=0) (actual time=0.027..0.027 rows=1 loops=1)
--         Index Cond: ((data ->> 'email'::text) = '947703b7@example.com'::text)


SELECT indexrelname                                 AS index_name,
       pg_size_pretty(pg_relation_size(indexrelid)) AS index_size -- 346 MB
FROM pg_stat_all_indexes
WHERE indexrelname = 'idx_json_data';


reindex index idx_json_data;

SELECT indexrelname                                 AS index_name,
       pg_size_pretty(pg_relation_size(indexrelid)) AS index_size -- 289 MB
FROM pg_stat_all_indexes
WHERE indexrelname = 'idx_json_data';


```

