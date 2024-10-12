
# №4 Блокировки

1. Создать таблицу accounts(id integer, amount numeric);

```
create table accounts
(
    id     integer primary key,
    amount numeric
);

insert into accounts (id, amount) values (1, 1000);
insert into accounts (id, amount) values (2, 2000);
insert into accounts (id, amount) values (3, 3000);
```

2. Добавить несколько записей и подключившись через 2 терминала добиться ситуации взаимоблокировки (deadlock).

```
term_1

begin;

update accounts set amount = 1001 where id = 1;
update accounts set amount = 2002 where id = 2;

```

```
term_2
begin;

update accounts set amount = 2001 where id = 2;
update accounts set amount = 1002 where id = 1;

```

3. Посмотреть логи и убедиться, что информация о дедлоке туда попала.

```
2024-10-12 20:50:18.543 MSK [92129] postgres@postgres ERROR:  deadlock detected
2024-10-12 20:50:18.543 MSK [92129] postgres@postgres DETAIL:  Process 92129 waits for ShareLock on transaction 969; blocked by process 92104.
        Process 92104 waits for ShareLock on transaction 968; blocked by process 92129.
        Process 92129: update accounts set amount = 2002 where id = 2
        Process 92104: update accounts set amount = 1002 where id = 1
2024-10-12 20:50:18.543 MSK [92129] postgres@postgres HINT:  See server log for query details.
2024-10-12 20:50:18.543 MSK [92129] postgres@postgres CONTEXT:  while updating tuple (0,11) in relation "accounts"
2024-10-12 20:50:18.543 MSK [92129] postgres@postgres STATEMENT:  update accounts set amount = 2002 where id = 2
2024-10-12 20:50:18.624 MSK [92129] postgres@postgres ERROR:  current transaction is aborted, commands ignored until end of transaction block
2024-10-12 20:50:18.624 MSK [92129] postgres@postgres STATEMENT:  select current_database() as a, current_schemas(false) as b
2024-10-12 20:50:18.656 MSK [92129] postgres@postgres ERROR:  current transaction is aborted, commands ignored until end of transaction block
2024-10-12 20:50:18.656 MSK [92129] postgres@postgres STATEMENT:  SHOW TRANSACTION ISOLATION LEVEL

```
