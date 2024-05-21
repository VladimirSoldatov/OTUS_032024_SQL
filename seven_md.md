1. Работаю в Windows. Нет прав администратора.
2. Использую VMWARE.
3. Скачал с сайта предустановленный образ с развернутой Ubuntu 23.10.
4. Создал виртуальную машину Linux без установки, подключив образ к систему в качестве HDD.
5. С помощью консоли Windows команды ssh-keygen -t ed25519 создал key pair.
6. С помощью команды type c:\users\vvsoldatov\ssh\id_rsa.pub | ssh osboxes@[имя сервера] -p [порт] "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys" перенес сертификат на сервер.
7. Проверил подключени, работает.
8. Поставил PostgreSQL 15, используя комманду sudo apt install postgresql.
9. Отредактировал pg_hba.conf заменив настройки локального подключения для пользователя postgres как trust вместо peer.
10. Перезапустил службу sudo systemctl restart postgresql.
11. sudo -u postgres psql -d testdb -- подключился к базе tempdb  как postgres
12. create schema testnm; -- создаем схему testnm
13. create table t1(c1 int); -- создаем в пространстве имен  testnm таблицу t1 со столбцом c1 тип данных INT
14. insert into t1(c1) values (1); -- вставляем данные
15. Создаем роль readonlу
16. Важно добавить доступ к схеме USAGE и право SELECT на таблице в схеме
17. Даем права readonly пользователю testread, фактически это включение пользователя в группу readonly
18. \du покажет, что пользователь testread включен в группу readonly
19. SELECT * FROM information_schema.role_table_grants where table_name = 't1' - проверка прав readonly
20. Далeе выходим из БД и входим под пользователем testread
21. Тестирование  наличие  доступа к данным в таблице t1 у пользвателя testread
22. Очевидны проблемы с доступом.
23. Первая из-за схемы - по умочанию поиск идет в схеме по написанию совпадающей с именем пользователя и потом public. При логировании под  пользователем testread поиск завершается поиском в схеме public. Очевидно, что у пользователя testread  нет прав на талицы в схеме public и получим ошибку прав.
24. Решается проблема путем создания таблицы с явным указанием схемы testnm.t1
25. Вторая менее очевидная и вытекает из первой. Поиск имени таблицы ведется сначала в схеме по написанию совпадающей с именем пользователя и потом public. Такой таблицы нет, требуется либо изменить SELECT либо перенастроить путь для поиска relation.
26. Третья связана с тем, что права на схему выданы до создания таблицы, решается это workaround перевыдачей прав на схему, либо permament: <br>0) \c testdb postgres<br>1) ALTER default privileges in SCHEMA testnm grant SELECT on TABLES to readonly; <br>2)
\c testdb testread;
27. t2(c1 integer); insert into t2 values (2); -- пройдут успешно, так по умолчанию будут создаваться в схеме public, на которую все пользователи имеют права
27. REVOKE CREATE on SCHEMA public FROM public; 
28. REVOKE ALL on DATABASE testdb FROM public;
29. Решается это только принудительным удалением прав у роли public, от которой идет не явное наследование для всех пользователей
30.create table t3(c1 integer); insert into t2 values (2); -- выдаст две ошибки. На попытку создания таблицы и попытку вставки в существующую public.t2;