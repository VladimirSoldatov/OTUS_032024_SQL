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

 <br>select locktype, pid, relation::regclass, virtualxid as virtxid,
 <br>transactionid as xid, mode, granted
 <br>from pg_locks
 <br>where pid <> pg_backend_pid()
 <br>order by pid, locktype;
 25. Видим три эксклюзивные блокировки

 |locktype| pid|relation|virtxid|xid|mode|granted|
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|virtualxid|614751||4/38||ExclusiveLock|t|
|virtualxid|614958||5/3||ExclusiveLock|t|
|virtualxid|614964||6/2||ExclusiveLock|t|

26. update my_blocks_table set data = 'Uzlovay' where id = 1; -- выполнили разный UPDATE в разных транзакциях


|#|locktype| pid|relation|virtxid|xid|mode|granted|
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|1|relation|614751|my_blocks_table|||RowExclusiveLock|t|
|2|transactionid|614751|||776|ExclusiveLock|t|
|3|transactionid|614751|||775|ShareLock|f|
|4|tuple|614751|my_blocks_table|||ExclusiveLock|t|
|5|virtualxid|614751||4/38||ExclusiveLock|t|
|6|relation|614958|my_blocks_table|||RowExclusiveLock|t|
|7|transactionid|614958|||777|ExclusiveLock|t|
|8|tuple|614958|my_blocks_table|||ExclusiveLock|f|
|9|virtualxid|614958||5/3||ExclusiveLock|t|
|10|relation|614964|my_blocks_table|||RowExclusiveLock|t|
|11|transactionid|614964|||775|ExclusiveLock|t|
|12|virtualxid|614964||6/3||ExclusiveLock|t|

- когда транзакции стартовали, им были присвоены виртуальные идентификаторы virtualxid => 4/38, 5/3, 6/3 (строки 5, 9, 12) и эти номера успешно (granted=t) заблокированы самими транзакциями в режиме исключительной блокировки (ExclusiveLock)

- транзакции дополнительно получили физические номера transactionid (xid) => 775, 776, 777 при попытке изменить данные командой update, и самостоятельно их удерживают (строки 2, 7, 11) в режиме ExclusiveLock

- также из-за команды update появились блокировки с типом relation - блокировки отношений (таблицы my_blocks_table), в режиме RowExclusiveLock (строки 1, 6, 10), выданы (granted=t), и самостоятельно удерживаются транзакциями

- вторая транзакция ожидает завершения первой транзакции - попыталась (granted=f) получить блокировку номера transactionid = 775 (первой транзакции) в режиме ShareLock (строка 3), и наложила (granted=t) блокировку tuple (блокировка версии строки) на обновляемую строку в режиме ExclusiveLock (строка 4)

- третья транзакция тоже попыталась (granted=f) получить блокировку tuple на обновляемую строку в режиме ExclusiveMode (строка 8), но неудачно, так как вторая транзакция уже наложила такую блокировку.

27. Для создания дедлока добавим столбоц с population INT для учета населения.

28. Значения населения для всех городов выставлено  1 000 0000
29. Открываем три транзации в разных окнах

|Транзакция 1|Транзакция 2| Транзакция 3|
|:-:|:-:|:-:|
|Добавляем 500 000 для ID = 1|||
||Добавляем 100 000 для ID = 2||
|||Уменьшаем 700 000 для ID = 3|
|Добавляем 10 000 для ID = 2|||
||Уменьшаем 700 000 для ID = 3||
|||Уменьшаем 30 000 для ID = 1|
|||commit;|
||commit;||
|commit;|||

30. В результате в третьем окне ошибка, с фиксацией взаимоь
ERROR:  deadlock detected
DETAIL:  Process 624320 waits for ShareLock on transaction 793; blocked by process 624315.<br>
Process 624315 waits for ShareLock on transaction 794; blocked by process 624317.<br>
Process 624317 waits for ShareLock on transaction 795; blocked by process 624320.<br>
HINT:  See server log for query details.<br>
CONTEXT:  while updating tuple (0,24) in relation "my_blocks_table"<br>

Последний вопрос может ли быть блокировка при UPDATE, SELECT без WHERE.  Мало вероятно.

Раз нельзя использовать WHERE остается только порядок SELECT при чтении с ORDER BY с начала и конца таблицы.
Либо данные будут одни браться последовательно из таблицы, а другие по указателям из индекса.

Тогда возможно попытка одновременного доступа к одной и той же записи.