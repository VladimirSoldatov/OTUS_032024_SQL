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

13. Скачал демонстрационную базу данных по ссылке  [https://edu.postgrespro.com/demo-big-en.zip]

13.1.  psql -f demo-big-en.sql -U postgres

14.1. Прямое соеддинение, оно же INNER JOIN. Возвращает только те строки, которые есть в обеих таблицах по ключу 

14.2. SELECT
<br>  f.flight_id
<br>, f.flight_no
<br>,  f.departure_airport
<br>, f.arrival_airport
<br>, s.seat_no 
<br>, s.fare_conditions 
<br> FROM flights f
<br> INNER join seats s on s.aircraft_code  = f.aircraft_code

15.1.  LEFT/RIGHT JOIN вернет то же самое, что и INNER JOIN плюс для не найденных хначений по ключу вернет недостающую информацию в виде NULL

15.2. SELECT
<br>  f.flight_id
<br>, f.flight_no
<br>,  f.departure_airport
<br>, f.arrival_airport
<br>, s.seat_no 
<br>, s.fare_conditions 
<br> FROM flights f
<br> LEFT join seats s on s.aircraft_code  = f.aircraft_code
<br> WHERE f.flight_id = 2880

15.3. Вернет 222 строки по конкретному рейсу

16.1. Полное пересечение

16.2. SELECT 
<br> f.flight_id
<br>, f.flight_no
<br>,  f.departure_airport
<br>, f.arrival_airport
<br>, s.seat_no 
<br>, s.fare_conditions 
<br>from flights f
<br>full join seats s on s.aircraft_code  = f.aircraft_code
<br> WHERE
<br> s.aircraft_code is null

16.3. Вернет ноль строк, так как все данные согласованы

17.1. Декартово произведение получаем использую ключевое слово CROSS

17.2. <br> SELECT 
<br> f.flight_id
<br>, f.flight_no
<br>,  f.departure_airport
<br>, f.arrival_airport
<br>, s.seat_no 
<br>, s.fare_conditions 
<br>from flights f
<br>cross join seats s

17.3. Вернет 297 млн. строк

18.1. Частичное декартово

18.2. Декартово произведение получаем использую ключевое слово CROSS

SELECT 
<br>f.flight_id
<br>, f.flight_no
<br>,  f.departure_airport
<br>, f.arrival_airport
<br>, s.seat_no 
<br>, s.fare_conditions 
<br>from flights f
<br> CROSS JOIN lateral
<br>(select seat_no, fare_conditions  from seats s2 where s2.aircraft_code =  f.aircraft_code) s

18.3. Вернет 16 млн. строк как и колчество рейсов