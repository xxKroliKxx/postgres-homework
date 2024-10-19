
# №6 Физическая и логическая репликация

```
Тесты с выключеной репликой 

postgres@homework:/home/kirill5$ pg_ctlcluster 16 slave stop

postgres@homework:/home/kirill5$ /usr/lib/postgresql/16/bin/pgbench -c 8 -j 4 -T 10 -f ~/workload2.sql -n -U postgres -p 5432 thai
number of transactions actually processed: 49564


postgres@homework:/home/kirill5$ /usr/lib/postgresql/16/bin/pgbench -c 8 -j 4 -T 10 -f ~/workload.sql -n -U postgres thai
number of transactions actually processed: 258558

----------------------------------
Тесты с включеной репликой

postgres@homework:/home/kirill5$ pg_ctlcluster 16 slave start
postgres@homework:/home/kirill5$ /usr/lib/postgresql/16/bin/pgbench -c 8 -j 4 -T 10 -f ~/workload2.sql -n -U postgres -p 5432 thai
number of transactions actually processed: 31484

postgres@homework:/home/kirill5$ /usr/lib/postgresql/16/bin/pgbench -c 8 -j 4 -T 10 -f ~/workload.sql -n -U postgres thai
number of transactions actually processed: 250779

-----------------------------------
Чтение с реплики на 5433 порту
postgres@homework:/home/kirill5$ /usr/lib/postgresql/16/bin/pgbench -c 8 -j 4 -T 10 -f ~/workload.sql -n -U postgres -p 5433 thai
number of transactions actually processed: 247904
```

