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
14. insert into t1(c1) values (1); -- вставлены
