
# №3 Система очистки, журнал предзаписи


1. Создать таблицу с текстовым полем и заполнить случайными или сгенерированными данным в размере 1 млн строк
```
create or replace function generate_rows() returns void
    security definer
    language plpgsql
as
$$
declare
    total_rows int := 1000000;
    i          int := 1;
begin
    while i <= total_rows
        loop
            insert into test_table (id, str_field)
            values (i, 'Value_' || i);
            i := i + 1;
        end loop;
end;
$$;

select generate_rows();

```
2. Посмотреть размер файла с таблицей

```
select pg_size_pretty(pg_relation_size('test_table')) as table_size;

49mb
```

3. 5 раз обновить все строчки и добавить к каждой строчке любой символ

```
create or replace function update_rows(_value text) returns void
    security definer
    language plpgsql
as
$$
begin
    update test_table
    set str_field = _value + id;
end;
$$;

select update_rows('1_value_');
select update_rows('2_value_');
select update_rows('3_value_');
select update_rows('4_value_');
select update_rows('5_value_');

```

4. Посмотреть количество мертвых строчек в таблице и когда последний раз приходил
   автовакуум

```
select n_dead_tup,
       last_autovacuum
from pg_stat_user_tables
where relname = 'test_table';

или

select
    pg_stat_get_dead_tuples(oid) AS dead_tuples,
    pg_stat_get_last_autovacuum_time(oid) AS last_autovacuum
from
    pg_class
where
    relname = 'test_table';

или

vacuum verbose test_table;

все запросы вернули одни и теже данные

Количество строк 2999822 - вместо 5кк - потому что пока апдейтил - вакум уже очистил часть)
```

5. Подождать некоторое время, проверяя, пришел ли автовакуум

``` ну да```

6. 5 раз обновить все строчки и добавить к каждой строчке любой символ
7. Посмотреть размер файла с таблицей

``` 
   он после предыдущих 5 обновлений стал стал 149мб
```

8. Отключить Автовакуум на конкретной таблице

```

alter table test_table set (autovacuum_enabled = false);

```

9. 10 раз обновить все строчки и добавить к каждой строчке любой символ
10. Посмотреть размер файла с таблицей

```

я допом 5 раз еще обновил, 10 занимает очень много времени)
размер стал 298 MB

```

11. Объясните полученный результат

```
При первом добавлении таблица весила 49mb
Затем во время update - postgres удалял исходные строки и добавлял новые, при этом паралельно работал автовакум, 
поэтому часть строк переиспользвала старые - сказал бы я, но понял что функция отработала в транзакции и он не мог переиспользовать их, до ее фиксации

Но не суть, когда автовакум отключил, он в целом не мог их переиспользовать и надобавлял новых

vacuum full test_table; 

после это размер таблицы опустился до 50mb - 1mb потому что длина строки после update изменилась

```

12. Не забудьте включить автовакуум)

Задание со *:
Написать анонимную процедуру, в которой в цикле 10 раз обновятся все строчки в искомой таблице.
Не забыть вывести номер шага цикла.

```

Я там функцию генерации и апдейта написал, это не зачтется?))

```
