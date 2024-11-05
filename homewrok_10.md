# №10 Оконные функции и полнотекстовый поиск

```

with _salary as (SELECT s.fk_employee,
                        s.amount,
                        s.amount -
                        LAG(amount, 1, 0) OVER (PARTITION BY fk_employee ORDER BY from_date) AS    previous_amount,
                        LAG(s.amount, 1, 0) OVER (PARTITION BY s.fk_employee ORDER BY s.from_date) previous
                 FROM salary s
                 ORDER BY s.fk_employee, s.from_date)
select e.id,
       e.first_name || ' ' || e.last_name as name,
       s.amount,
       s.previous,
       s.previous_amount
from employee e
         left join _salary s on e.id = s.fk_employee

```

