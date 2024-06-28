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
11. CREATE DATABASE my_blocks_db;  -- создал базу данных для проверки взаимоблокировок;
12. \c my_blocks_db -- подключаемся к данной БД
13. show log_lock_waits;  -- получаем сведения о текущем уровне логирования долгих запросов

|log_lock_waits|
|:-:|
|off|

14. show deadlock_timeout; -- получаем текущее значение таймаута для лоирования

|deadlock_timeout|
|:-:|
|1s|

15. select name, context from pg_settings where name in ('log_lock_waits', 'deadlock_timeout'); -- смотрим контекст данных настроек

| name|context|
|:-:|:-:|
|deadlock_timeout|superuser|
|log_lock_waits|superuser|

16. Данные настройки не для установить для сеанса, потребуется изменение файлов конфигурации и перезапуск службы
17. Меняем настройки
alter system set deadlock_timeout = 200;  
alter system set log_lock_waits = on; 
18. sudo systemctl restart postgresql -- перезапускаем службу
19. show log_lock_waits;  -- получаем сведения о текущем уровне логирования долгих запросов

|log_lock_waits|
|:-:|
|on|

20. show deadlock_timeout; -- получаем текущее значение таймаута для лоирования

|deadlock_timeout|
|:-:|
|200s|
21. Все хорошо
22. create database my_blocks_db; -- создаем новую БД под блокировки
23. create my_blocks_table(int ID, data varchar(50)); -- создаем таблицу с двумя столбцами
24. insert into my_blocks_table values (1, 'Moscow'),(2, 'New York'), (3, 'Sydney'); Добавляем  данные