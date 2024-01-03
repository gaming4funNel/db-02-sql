# Домашнее задание к занятию "SQL" - Иванов Игорь

## Задача 1

Используя Docker, поднимите инстанс PostgreSQL (версию 12) c 2 volume, 
в который будут складываться данные БД и бэкапы.

Приведите получившуюся команду или docker-compose-манифест.

```
version: '3.8'

services:
  postgres:
    image: postgres:12
    container_name: postgres12
    restart: always
    ports:
      - 5432:5432
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: 1234qwer!@
    volumes:
      - pg_data:/var/lib/postgresql/data
      - pg_backups:/backups

volumes:
  pg_data:
  pg_backups:
```

## Задача 2

В БД из задачи 1: 

- создайте пользователя test-admin-user и БД test_db;
- в БД test_db создайте таблицу orders и clients (спeцификация таблиц ниже);
- предоставьте привилегии на все операции пользователю test-admin-user на таблицы БД test_db;
- создайте пользователя test-simple-user;
- предоставьте пользователю test-simple-user права на SELECT/INSERT/UPDATE/DELETE этих таблиц БД test_db.

Таблица orders:

- id (serial primary key);
- наименование (string);
- цена (integer).

Таблица clients:

- id (serial primary key);
- фамилия (string);
- страна проживания (string, index);
- заказ (foreign key orders).

Приведите:

- итоговый список БД после выполнения пунктов выше;

![pgdb](https://github.com/gaming4funNel/db-02-sql/blob/main/img/pgdb1.png)

- описание таблиц (describe);

![pgdb](https://github.com/gaming4funNel/db-02-sql/blob/main/img/pgdb2.png)

- SQL-запрос для выдачи списка пользователей с правами над таблицами test_db;

```
SELECT grantee, table_name, privilege_type
FROM information_schema.table_privileges
WHERE table_catalog = 'test_db';
```

- список пользователей с правами над таблицами test_db.

![pgdb](https://github.com/gaming4funNel/db-02-sql/blob/main/img/pgdb3.png)

## Задача 3

Используя SQL-синтаксис, наполните таблицы следующими тестовыми данными:

Таблица orders

|Наименование|цена|
|------------|----|
|Шоколад| 10 |
|Принтер| 3000 |
|Книга| 500 |
|Монитор| 7000|
|Гитара| 4000|

Таблица clients

|ФИО|Страна проживания|
|------------|----|
|Иванов Иван Иванович| USA |
|Петров Петр Петрович| Canada |
|Иоганн Себастьян Бах| Japan |
|Ронни Джеймс Дио| Russia|
|Ritchie Blackmore| Russia|

Используя SQL-синтаксис:
- вычислите количество записей для каждой таблицы.

Приведите в ответе:
- запросы,
- результаты их выполнения.

```
INSERT INTO orders (name, price) VALUES
('Шоколад', 10),
('Принтер', 3000),
('Книга', 500),
('Монитор', 7000),
('Гитара', 4000);
```

![pgdb](https://github.com/gaming4funNel/db-02-sql/blob/main/img/pgdb4.png)

```
INSERT INTO clients (last_name, country) VALUES
('Иванов Иван Иванович', 'USA'),
('Петров Петр Петрович', 'Canada'),
('Иоганн Себастьян Бах', 'Japan'),
('Ронни Джеймс Дио', 'Russia'),
('Ritchie Blackmore', 'Russia');
```

![pgdb](https://github.com/gaming4funNel/db-02-sql/blob/main/img/pgdb5.png)

```
SELECT COUNT(*) FROM orders;
SELECT COUNT(*) FROM clients;
```
![pgdb](https://github.com/gaming4funNel/db-02-sql/blob/main/img/pgdb6.png)

## Задача 4

Часть пользователей из таблицы clients решили оформить заказы из таблицы orders.

Используя foreign keys, свяжите записи из таблиц, согласно таблице:

|ФИО|Заказ|
|------------|----|
|Иванов Иван Иванович| Книга |
|Петров Петр Петрович| Монитор |
|Иоганн Себастьян Бах| Гитара |

Приведите SQL-запросы для выполнения этих операций.

```
UPDATE clients SET order_id = (SELECT id FROM orders WHERE name = 'Книга') WHERE last_name = 'Иванов Иван Иванович';
UPDATE clients SET order_id = (SELECT id FROM orders WHERE name = 'Монитор') WHERE last_name = 'Петров Петр Петрович';
UPDATE clients SET order_id = (SELECT id FROM orders WHERE name = 'Гитара') WHERE last_name = 'Иоганн Себастьян Бах';
```

Приведите SQL-запрос для выдачи всех пользователей, которые совершили заказ, а также вывод этого запроса.

```
SELECT clients.last_name, clients.country, orders.name AS заказ
FROM clients
INNER JOIN orders ON clients.order_id = orders.id
WHERE clients.order_id IS NOT NULL;
```

Подсказка: используйте директиву `UPDATE`.

## Задача 5

Получите полную информацию по выполнению запроса выдачи всех пользователей из задачи 4 
(используя директиву EXPLAIN).

Приведите получившийся результат и объясните, что значат полученные значения.

![pgdb](https://github.com/gaming4funNel/db-02-sql/blob/main/img/pgdb7.png)

Hash Join указывает на использование хеш-соединения. Hash Cond показывает условие, по которому производится соединение. Seq Scan указывает на последовательное сканирование таблицы. Filter показывает условие фильтрации.
Каждая строка представляет один шаг выполнения запроса. Стоимость (cost) и оценка количества строк (rows) могут быть полезными для оптимизации запроса.

## Задача 6

Создайте бэкап БД test_db и поместите его в volume, предназначенный для бэкапов (см. задачу 1).

```
docker exec -it postgres12 bash
pg_dump -U admin test_db > /backups/test_db_backup.sql
```

Остановите контейнер с PostgreSQL, но не удаляйте volumes.

```
docker stop postgres12
```

Поднимите новый пустой контейнер с PostgreSQL.

```
docker run -d \
  --name test_db \
  -v /var/lib/docker/volumes/postgres_pg_backups/_data:/backups \
  -e POSTGRES_DB=test_db \
  -e POSTGRES_USER=admin \
  -e POSTGRES_PASSWORD=1234qwer \
  postgres:12
```

Восстановите БД test_db в новом контейнере.

```
docker exec -it test_db bash
psql -U admin test_db < backups/test_db_backup.sql
```

Приведите список операций, который вы применяли для бэкапа данных и восстановления. 

---

### Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---