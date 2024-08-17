1. Работаю в Windows. Нет прав администратора.
2. Использую VMWARE.
3. Скачал с сайта предустановленный образ с развернутой Ubuntu 20.04.6 LTS
4. Создал виртуальную машину Linux без установки, подключив образ к систему в качестве HDD.
5. С помощью консоли Windows команды ssh-keygen -t rsa создал key pair.
6. С помощью команды type c:\users\[имя_пользователя]\ssh\id_rsa.pub | ssh osboxes@[имя сервера] -p [порт] "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys" перенес сертификат на сервер.
7. Проверил подключени, работает.
8. Поставил PostgreSQL 16, используя комманду sudo apt install postgresql-16, после того как добавил репозитарий и публичный ключ.
9. Отредактировал pg_hba.conf заменив настройки локального подключения для пользователя postgres как trust вместо peer.
10. Перезапустил службу sudo systemctl restart postgresql.
11. Подключился как пользователь postgres - sudo psql postgres -U postgres и создал тестовую базу demo_otus.\
12. Переподключился на тестовую базу - \c demo_otus.
<center>
<table>
<tr><td>sudo -u postgres pg_lsclusters</td></tr>
<tr><td>Ver</td><td>Cluster Port </td><td>Status Owner</td><td>Data directory</td><td>Log file</td></tr>
<tr><td>16</td><td>main</td><td>5432</td><td>online</td><td>postgres</td><td>/var/lib/postgresql/16/main /var/log/postgresql/postgresql-16-main.log</td></tr>
</table>
</center>

13. С сайта PostgresPro взял демонстрационную базу перелетов. Из трех вариантов взял наиболее большую по размеру.
14. Разагривировал архив и загрузил БД.
15. Все таблицы содержать необходимые индексы, поэтому я решил создать новую таблицу без индексов
16. <b> create table bookings.boarding_passes_clone as select * 
from bookings.boarding_passes</b>
17. Размер таблицы 1.1 Gb, что составляет почти 8 млн. строк.
18. Таблица bookings.boarding_passes_clone не имеет индексов
19. explain analyze select * from bookings.boarding_passes_clone;
20. Seq Scan on boarding_passes_clone  (cost=0.00..137563.08 rows=7925908 width=25) (actual time=0.009..2715.406<br> rows=7925812 loops=1)<br>
Planning Time: 0.042 ms<br>
Execution Time: 2914.195 ms<br>
21. explain select * from bookings.boarding_passes_clone where ticket_no = '0005432268510'
22. Gather  (cost=1000.00..100585.07 rows=3 width=25)<br>
  Workers Planned: 2<br>
  ->  Parallel Seq Scan on boarding_passes_clone  (cost=0.00..99584.77 rows=1 width=25)<br>
        Filter: (ticket_no = '0005432268510'::bpchar)<br>
    Execution Time: 334.423 ms<br>
23. Как мы видим в плане выполнения запроса происходит последовательное чтение таблицы с фильтрацией по значению поля ticket_no. Время поиска по уже прогретым данным 334 мс.
24. create index ticket_no_index ON bookings.boarding_passes USING btree (ticket_no);
25. Index Scan using ticket_no_index on boarding_passes_clone  (cost=0.43..16.48 rows=3 width=25) (actual time=0.699..0.700 rows=1 loops=1)<br>
  Index Cond: (ticket_no = '0005432268510'::bpchar)<br>
Planning Time: 0.074 ms<br>
Execution Time: 0.713 ms<br>
25. После создания индекса используется сканирование индекса
27. Но наиблее эффективное использование индекса происходит при получении данных напрямую из индекса, так сам индекс весит меньше чем таблица
28. Index Only Scan using ticket_no_index on boarding_passes_clone  (cost=0.43..8.48 rows=3 width=14) (actual time=0.<br>043..0.044 rows=1 loops=1)<br>
  Index Cond: (ticket_no = '0005432268510'::bpchar)<br>
  Heap Fetches: 0<br>
Planning Time: 0.046 ms<br>
Execution Time: 0.055 ms<br>
29. Наиболее эффективным решение являтся создание составного индекса из нескольх полей (покрывающего индекса)
30. Создание полнотекстового индекса
31. explain analyze <br>
SELECT ticket_no, book_ref, passenger_id, passenger_name, contact_data<br>
FROM bookings.tickets where <br>
to_tsvector(passenger_name) @@ to_tsquery('MIKHAIL');<br>
32. Execution Time: 6089.024 ms<br>
33. Filter: (to_tsvector(passenger_name) @@ to_tsquery('MIKHAIL'::text))
34. create index pas_name_index on bookings.tickets using gin (to_tsvector('english',passenger_name));
35. Так как для создания требуется  IMMUTABLE функция, умеющая работать с текстом
36. explain analyze <br>
SELECT ticket_no, book_ref, passenger_id, passenger_name, contact_data <br>
FROM bookings.tickets<br>
where to_tsvector('english', passenger_name) @@ to_tsquery('MIKHAIL');<br>
37. Workers Launched: 2<br>
  ->  Parallel Bitmap Heap Scan on tickets  (cost=111.11..34981.68 rows=6145 width=104) (actual time=4.820..73.091<br> rows=14163 loops=3)<br>
        Recheck Cond: (to_tsvector('english'::regconfig, passenger_name) @@ to_tsquery('MIKHAIL'::text))<br>
        Heap Blocks: exact=13204<br>
        ->  Bitmap Index Scan on pas_name_index  (cost=0.00..107.42 rows=14749 width=0) (actual time=9.901..9.902 <br>rows=42488 loops=1)<br>
              Index Cond: (to_tsvector('english'::regconfig, passenger_name) @@ to_tsquery('MIKHAIL'::text))<br>
Planning Time: 0.109 ms<br>
Execution Time: 124.138 ms<br>
38. Индексы могут быть также созданы с применением какой либо функции как на примере с GIN индексом, там и например для использования функции lover, upper или иной функции. CREATE INDEX  ON bookings.tickets(lower(passenger_name));
39. create index pass_id_index on bookings.tickets(cast (replace(passenger_id,' ','') as BIGINT));
40. Запрос explain analyze SELECT ticket_no, book_ref, passenger_id, passenger_name, contact_data <br>
FROM bookings.tickets<>
where cast (replace(passenger_id,' ','') as BIGINT) = 9864114954<>
41. Index Cond: ((replace((passenger_id)::text, ' '::text, ''::text))::bigint = '9864114954'::bigint)
42. Execution Time: 0.046 ms
43. Прямой поиск explain analyze SELECT ticket_no, book_ref, passenger_id, passenger_name, contact_data
FROM bookings.tickets
where passenger_id  = '9864 114954'
44. Execution Time: 517.000 ms
45. CREATE UNIQUE INDEX boarding_passes_flight_id_seat_no_key ON bookings.boarding_passes USING btree (flight_id, seat_no);
46. Создали уникальный индекс для двух полей. Явлется логичнымЮ если мы хотим получить данные для конкретного пассажира в рамкаъ рейса.
