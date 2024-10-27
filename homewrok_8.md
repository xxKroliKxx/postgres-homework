
# №8 Языки программирования и пользовательские типы данных

```
CREATE TABLE sales (
                       id SERIAL PRIMARY KEY,
                       sale_date DATE,
                       amount NUMERIC
);

INSERT INTO sales (sale_date, amount)
VALUES
    ('2023-01-15', 100.50),
    ('2023-02-10', 200.75),
    ('2023-03-05', 300.25),
    ('2023-04-20', 150.00),
    ('2023-05-18', 250.50),
    ('2023-06-22', 175.75),
    ('2023-07-14', 225.25),
    ('2023-08-30', 275.00),
    ('2023-09-10', 325.50),
    ('2023-10-05', 350.00),
    ('2023-11-11', 400.25),
    ('2023-12-25', 450.75);

CREATE OR REPLACE FUNCTION get_year_third(month_num NUMERIC)
    RETURNS VARCHAR AS $$
DECLARE
    mod_val INT;
BEGIN
    IF month_num IS NULL THEN
        RETURN NULL;
    END IF;

    mod_val := month_num % 12;

    RETURN CASE
               WHEN mod_val >= 1 AND mod_val <= 4 THEN 'Первая треть'
               WHEN mod_val >= 5 AND mod_val <= 8 THEN 'Вторая треть'
               WHEN mod_val >= 9 OR mod_val = 0 THEN 'Третья треть'
               ELSE 'Некорректный месяц'
        END;
END;
$$ LANGUAGE plpgsql;

CREATE OR REPLACE FUNCTION get_year_third(sale sales)
    RETURNS VARCHAR AS $$
DECLARE
    mod_val INT;
BEGIN
    IF sale.sale_date IS NULL THEN
        RETURN NULL;
    END IF;

    mod_val := EXTRACT(MONTH FROM sale.sale_date) % 12;

    RETURN CASE
               WHEN mod_val >= 1 AND mod_val <= 4 THEN 'Первая треть'
               WHEN mod_val >= 5 AND mod_val <= 8 THEN 'Вторая треть'
               WHEN mod_val >= 9 OR mod_val = 0 THEN 'Третья треть'
               ELSE 'Некорректный месяц'
        END;
END;
$$ LANGUAGE plpgsql;

SELECT
    s.id,
    s.sale_date,
    s.amount,
    get_year_third(EXTRACT(MONTH FROM s.sale_date)) AS year_third_1,
    get_year_third(s) AS year_third_2
FROM
    sales s;


```

