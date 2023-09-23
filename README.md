Roman Teterevlev

Индексы

### Задание 1

Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.

### Ответ:

```sql
select ROUND(SUM(index_length)*100/SUM(data_length)) 'size index in %' from INFORMATION_SCHEMA.TABLES
```


### Задание 2

Выполните explain analyze следующего запроса:
```sql
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
```
- перечислите узкие места;
- оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы.

### Ответ:

При текущем запросе результат:


Столбец f.title, таблицы film и inventory в функции увеличивают время обработки запроса.
Добавим индекс:
```
CREATE index payment_date on payment(payment_date);
```
Приведем к виду:
```sql
select concat(c.last_name, ' ', c.first_name), sum(p.amount)
from payment p
inner join rental r on r.rental_date = p.payment_date
inner join customer c on c.customer_id = r.customer_id 
where p.payment_date between '2005-07-30 00:00:00' and '2005-07-30 23:59:59'
group by c.last_name, c.first_name, c.customer_id
```

