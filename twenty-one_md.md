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

14. Создаю таблицу с идентичныыми полями оригиналу<br><br>

create table <br>
boarding_passes_hash(ticket_no bpchar(13), flight_id int, boarding_no <br>int, seat_no varchar(4))<br>
partition by hash(flight_id);<br>

15. Создаем  партиции 10 штук

CREATE TABLE boarding_passes_hash_0 PARTITION OF boarding_passes_hash <br>
    FOR VALUES WITH (MODULUS 10, REMAINDER 0);<br><br>
CREATE TABLE boarding_passes_hash_1 PARTITION OF boarding_passes_hash<br>
    FOR VALUES WITH (MODULUS 10, REMAINDER 1);<br><br>
CREATE TABLE boarding_passes_hash_2 PARTITION OF boarding_passes_hash<br>
    FOR VALUES WITH (MODULUS 10, REMAINDER 2);<br><br>
CREATE TABLE boarding_passes_hash_3 PARTITION OF boarding_passes_hash<br>
    FOR VALUES WITH (MODULUS 10, REMAINDER 3);<br><br>
CREATE TABLE boarding_passes_hash_4 PARTITION OF boarding_passes_hash<br>
    FOR VALUES WITH (MODULUS 10, REMAINDER 4);<br><br>
CREATE TABLE boarding_passes_hash_5 PARTITION OF boarding_passes_hash<br>
    FOR VALUES WITH (MODULUS 10, REMAINDER 5);<br><br>
CREATE TABLE boarding_passes_hash_6 PARTITION OF boarding_passes_hash<br>
    FOR VALUES WITH (MODULUS 10, REMAINDER 6);<br><br>
CREATE TABLE boarding_passes_hash_7 PARTITION OF boarding_passes_hash<br>
    FOR VALUES WITH (MODULUS 10, REMAINDER 7);<br><br>
CREATE TABLE boarding_passes_hash_8 PARTITION OF boarding_passes_hash<br>
    FOR VALUES WITH (MODULUS 10, REMAINDER 8);<br><br>
CREATE TABLE boarding_passes_hash_9 PARTITION OF boarding_passes_hash<br>
    FOR VALUES WITH (MODULUS 10, REMAINDER 9);<br>
    
16. Переностим данные


insert into boarding_passes_hash<br>
select * from boarding_passes;<br>

17. Проверяем соответствие строк.

select count(*) from boarding_passes<br>
union ALL<br>
select count(*) from boarding_passes_hash;<br>

18. Далее можно старую таблицу удалить

19. Текущую переименовать