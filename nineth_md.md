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
11. SHOW checkpoint_timeout;

| checkpoint_timeout |
|:-:|
|5min|
12. select name, setting, unit, category, shor_desc, context, from pg_settings where name = 'checkpoint_timeout';

|        name        | setting | unit |           category            |                        short_desc                        | context |
|:-:|:-:|:-:|:-:|:-:|:-:|
| checkpoint_timeout | 300     | s    | Write-Ahead Log / Checkpoints | Sets the maximum time between automatic WAL checkpoints. | sighup  |
13. Контекст signup означает, что для применения параметра нет необходимости перезапускать сервер, достаточно перечитать файл конфигурации
13. show log_checkpoints;

| log_checkpoints |
|:-:|
|on|
14. В данном случае, логирование checkpoint включено и включать его нет необходимости
15. select pg_current_wal_insert_lsn(); -- получаем текущий LSN;

| pg_current_wal_insert_ls |
|:-:|
|3/5808FFD0|

16. select checkpoints_timed from pg_stat_bgwriter; -- количество имеющихся чекпоинтов до тестирования

| checkpoints_timed |
|:-:|
|469|
17. LSN до тестирования

| pg_current_wal_insert_ls |
|:-:|
|3/5A6BD608|

18. Количество чекпоинтов после тестирования

| checkpoints_timed |
|:-:|
|470|

| Размер данных между чекпонтами |
|:-:|
|40031800|

19. Размер в мегабайтах 38,177 Mb

20. show checkpoint_completion_target; -- сверяемся с файлом логов

- 2024-06-05 06:28:05.878 EDT [254291] postgres@demo_otus STATEMENT:  select name, setting, unit, category, shor_desc, context, from pg_settings where name = 'checkpoint_timeout';<br>
- 2024-06-05 06:28:11.635 EDT [254291] postgres@demo_otus STATEMENT:  select name, setting, unit, category, shor_desc, context from pg_settings where name = 'checkpoint_timeout';<br>
- 2024-06-05 06:47:35.241 EDT [1464] LOG:  checkpoint starting: time <br>
- 2024-06-05 06:51:32.371 EDT [1464] LOG:  checkpoint complete: wrote 2395 buffers (1.8%); 0 WAL file(s) added, 0 removed, 2 recycled; write=237.122 s, sync=0.001 s, total=237.130 s; sync files=25, longest=0.001 s, average=0.001 s;<br>
- distance=39120 kB, estimate=39120 kB

21. Файлы подтверждаю создание одного checkpoint
22. SHOW synchronous_commit;

| synchronous_commit |
|:-:|
|on|

23. Не видим смены парамтера и поэтому перезапускаем кластер

24. pgbench -U postgres -i postgres создал объекты для тестирования<br>
25. osboxes@osboxes:~$ pgbench -c8 -P 6 -T 60 -U postgres postgres<br>

- osboxes@osboxes:~$ pgbench -c8 -P 6 -T 60 -U postgres postgres<br>
- pgbench (15.7 (Ubuntu 15.7-0ubuntu0.23.10.1))<br>
- starting vacuum...end.<br>
- progress: 6.0 s, 870.8 tps, lat 9.137 ms stddev 4.330, 0 failed<br>
- progress: 12.0 s, 902.7 tps, lat 8.861 ms stddev 4.198, 0 failed<br>
- progress: 18.0 s, 903.5 tps, lat 8.843 ms stddev 4.342, 0 failed<br>
- progress: 24.0 s, 820.8 tps, lat 9.753 ms stddev 4.576, 0 failed<br>
- progress: 30.0 s, 790.3 tps, lat 10.124 ms stddev 4.916, 0 failed<br>
- progress: 36.0 s, 830.5 tps, lat 9.626 ms stddev 4.449, 0 failed<br>
- progress: 42.0 s, 841.0 tps, lat 9.514 ms stddev 4.364, 0 failed<br>
- progress: 48.0 s, 874.8 tps, lat 9.147 ms stddev 4.581, 0 failed<br>
- progress: 54.0 s, 904.3 tps, lat 8.849 ms stddev 4.175, 0 failed<br>
- progress: 60.0 s, 891.5 tps, lat 8.970 ms stddev 4.047, 0 failed<br>
- transaction type: <builtin: TPC-B (sort of)><br>
- scaling factor: 1<br>
- query mode: simple<br>
- number of clients: 8<br>
- number of threads: 1<br>
- maximum number of tries: 1<br>
- duration: 60 s<br>
- number of transactions actually processed: 51790<br>
- number of failed transactions: 0 (0.000%)<br>
- latency average = 9.265 ms<br>
- latency stddev = 4.419 ms<br>
- initial connection time = 28.813 ms<br>
- tps = 863.021852 (without initial connection time)<br>

26. ALTER SYSTEM SET synchronous_commit = off 

28. select name, setting, unit, category, short_desc, context from pg_settings where name = 'synchronous_commit';

29. Контекст user и значение неизменно ON


|        name        | setting | unit |           category            |                        short_desc                        | context |
|:-:|:-:|:-:|:-:|:-:|:-:|
| synchronous_commit | on      |      | Write-Ahead Log / Settings | Sets the current transaction's synchronization level. | user    |


30. sudo systemctl restart postgresql -- рестарт кластера
31. pgbench -U postgres -i postgres -инициализация

- osboxes@osboxes:~$ pgbench -c8 -P 6 -T 60 -U postgres postgres<br>
- pgbench (15.7 (Ubuntu 15.7-0ubuntu0.23.10.1))<br>
- starting vacuum...end.<br>
- progress: 6.0 s, 1672.2 tps, lat 4.753 ms stddev 3.323, 0 failed<br>
- progress: 12.0 s, 1682.8 tps, lat 4.754 ms stddev 3.154, 0 failed<br>
- progress: 18.0 s, 1650.9 tps, lat 4.841 ms stddev 3.219, 0 failed<br>
- progress: 24.0 s, 1693.1 tps, lat 4.727 ms stddev 3.134, 0 failed<br>
- progress: 30.0 s, 1691.5 tps, lat 4.729 ms stddev 3.158, 0 failed<br>
- progress: 36.0 s, 1692.6 tps, lat 4.724 ms stddev 3.279, 0 failed<br>
- progress: 42.0 s, 1636.2 tps, lat 4.891 ms stddev 3.509, 0 failed<br>
- progress: 48.0 s, 1721.1 tps, lat 4.648 ms stddev 3.292, 0 failed<br>
- progress: 54.0 s, 1763.5 tps, lat 4.537 ms stddev 3.157, 0 failed<br>
- progress: 60.0 s, 1746.5 tps, lat 4.580 ms stddev 3.173, 0 failed<br>
- transaction type: <builtin: TPC-B (sort of)><br>
- scaling factor: 1<br>
- query mode: simple<br>
- number of clients: 8<br>
- number of threads: 1<br>
- maximum number of tries: 1<br>
- duration: 60 s<br>
- number of transactions actually processed: 101711
- number of failed transactions: 0 (0.000%)
- latency average = 4.717 ms
- latency stddev = 3.246 ms
- initial connection time = 36.786 ms
- tps = 1695.236592 (without initial connection time)

32. Производительность выросла пости в 4 раза
33. sudo pg_createcluster 15 second --data-checksums

34. osboxes@osboxes:~$ sudo -u postgres pg_lsclusters
- Ver Cluster Port Status Owner    Data directory                Log file
- 15  main    5432 online postgres /mnt/data/15/main             /var/log/postgresql/postgresql-15-main.log
- 15  second  5433 online postgres /var/lib/postgresql/15/second /var/log/postgresql/postgresql-15-second.log
35. sudo -u postgres psql -p 5433
36. show data_checksums; -- сверяемся, что все ок
37. create table users (id int, "name" text);
38. insert into test (id, "name") values (1, 'Ivan'), (2, 'Masha');
39. select pg_relation_filepath('users');
40. 

| pg_relation_filepath |
|:-:|
|base/5/16388|

41. sudo dd if=/dev/zero of=/var/lib/postgresql/15/second/base/5/16388 oflag=dsync conv=notrunc bs=1 count=8
- 8+0 records in
- 8+0 records out
- 8 bytes copied, 0.00747059 s, 1.1 kB/s 
42. sudo pg_ctlcluster 11 second start -- запуск кластера
43. postgres=# select * from users;
- WARNING:  page verification failed, calculated checksum 56762 but expected 31096
- ERROR:  invalid page in block 0 of relation base/5/16388
43. show ignore_checksum_failure;

| ignore_checksum_failure |
|:-:|
|off|

44. postgres=# select context from pg_settings where name = 'ignore_checksum_failure';

|  context |
|:-:|
|superuser|

45. То есть действует в рамках сеанса
46. show ignore_checksum_failure;


| ignore_checksum_failure |
|:-:|
|on|

47. SELECT

WARNING:  page verification failed, calculated checksum 56762 but expected 31096
|id |name|
|:-:|:-:|
|1| Ivan|
|2| Masha|