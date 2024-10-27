
# №7 Индексы и их слабые стороны

3. Проверить скорость выполнения сложного запроса (приложен в конце файла скриптов)

```
Planning Time: 1.202 ms
Execution Time: 1511.716 ms

```

4. Навесить индексы на внешние ключ

```
CREATE INDEX idx_ride_fkschedule ON book.ride(fkschedule);
CREATE INDEX idx_ride_fkbus ON book.ride(fkbus);
CREATE INDEX idx_schedule_fkroute ON book.schedule(fkroute);
CREATE INDEX idx_busroute_fkbusstationfrom ON book.busroute(fkbusstationfrom);

```


5. Проверить, помогли ли индексы на внешние ключи ускориться

```

нет, план запроса не использует их


```

