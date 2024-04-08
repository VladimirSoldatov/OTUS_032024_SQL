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
11. Подключился как пользователь postgres - sudo psql postgres -U postgres и создал тестовую базу demo_otus.\
12. Переподключился на тестовую базу - \c demo_otus.
13. Открыл две консоли для подключения к БД и поключился к БД demo _otus под postgres.
14. Выполнил \set AUTOCOMMIT off в первом окне.
15. Выполнил на создание таблицы с данными из задания. Данные закомитчены, а значит отображатся в обеих сессиях.
16. Режим изоляции по умолчанию и на момент подключения READ COMMITED
17. В каждой сессии начат своя транзакция.
demo_otus=*# select* from persons;

|id|first_name|second_name|
|:-:|:-:|:-:|
|1|ivan|ivanov|
|2|petr|petrov|

18. insert into persons(first_name, second_name) values('sergey', 'sergeev');
19. До комита вторая сессия не видит изменения от первой сессии. Причина уровень изоляции - READ COMMITED
20. COMMIT 
21. После комита в первой сессии изменения стали заметны и во второй.

|id|first_name|second_name|
|:-:|:-:|:-:|
|1|ivan|ivanov|
|2|petr|petrov|
|3|sergey|sergeev|

22. После чего транзакции завершены.
23. В обеих сессия открыл транзации с уровнем изоляции repeatable read.
24. В первой сессии выполнен запрос insert into persons(first_name, second_name) values('sveta', 'svetova');
25. Результат аналогичен предыдущему - изменения не видны во второй сессии без комита.
26. После выполнения COMMIT в первой сессии данные изменения стали заметны и во второй сессии.

|id|first_name|second_name|
|:-:|:-:|:-:|
|1|ivan|ivanov|
|2|petr|petrov|
|3|sergey|sergeev|
|4|sveta|svetova|

27. Вывод: В Postgres в приниципе не возможно "грязное чтения", так доступны для чтения только данные после COMMIT.

 