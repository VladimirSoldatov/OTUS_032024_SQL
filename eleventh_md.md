1. Работаю в Windows. Нет прав администратора.
2. На этот раз вместо VMWARE использую облако от Яндекс, что ради опыта.
3. Зарегистрировался на Yandex.Cloud и создал  ВМ с Ubuntu 24.04
4. Создал ssh-ключ через ssh-keygen -t ed25519
5. Через  команду type вывел содержимое pub-файла на консоль и скопировал его в Yandex Console при созданири
6. Установил Yandex CLI командой 
- iex (New-Object System.Net.WebClient).DownloadString('https://storage.yandexcloud.net/yandexcloud-yc/install.ps1')
7. yc compute ssh --name benchdemo --folder-id b1g2r4f9roncvsuh8j8d
8. Установил соединение с удаленной ВМ
9. Установил postgresql командой sudo apt install postgresql postgresql-client
10. Вывод на экран

<center>
<table>
<tr><td>sudo -u postgres pg_lsclusters</td></tr>
<tr><td>Ver</td><td>Cluster Port </td><td>Status Owner</td><td>Data directory</td><td>Log file</td></tr>
<tr><td>16</td><td>main</td><td>5432</td><td>online</td><td>postgres</td><td>/var/lib/postgresql/16/main /var/log/postgresql/postgresql-16-main.log</td></tr>
</table>
</center>

11. Установка sysbenc
12. curl -s https://packagecloud.io/install/repositories/akopytov/sysbench/script.deb.sh | sudo bash
13. sudo apt -y install sysbench
14.Клонируем репозиторий - git clone https://github.com/Percona-Lab/sysbench-tpcc 
15. sudo -U postgres psql
16. show hba_file; 
17. \# IPv4 local connections:<br>
host    all             all             127.0.0.1/32            trust
18. ./tpcc.lua --pgsql-host=127.0.0.1 --pgsql-port=5432 --pgsql-user=postgres --pgsql-db=bench_test --time=120 --report-interval=1  --tables=10 --scale=10 --use_fk=0 --trx_level=RC --db-driver=pgsql prepare

19. select datname, pg_size_pretty(pg_database_size(datname)) as "Size" from pg_stat_database;



|datname|Size|
|:-:|:-:|
|postgres|7500 kB|
|bench_test|10 GB|
|template1|7564 kB|
|template0|7345 kB|
|(5 rows)||

20. ./tpcc.lua --pgsql-host=127.0.0.1 --pgsql-port=5432 --pgsql-user=postgres --pgsql-db=bench_test --time=600 --report-interval=1 --tables=10 --scale=10 --use_fk=0 --trx_level=RC --db-driver=pgsql run

SQL statistics:<br>
 <table>
 <th>queries performed:</th>
 <tr><td><td>read:</td><td>10548</td></tr>
 <tr><td></td><td> write: </td><td>10864</td></tr>
 <tr><td></td><td> other: </td><td> 1624</td></tr>
 <tr><td><td> total: </td><td> 23036M</td></tr>
 <tr><td>transactions: </td><td> 811 (1.35 per sec.)</td></tr>
<tr><td><td>queries: </td><td> 23036 (38.34 per sec.)</td></tr>
 <tr><td><td>ignored errors: </td><td> 5 (0.01 per sec.)</td></tr>
 <tr><td><td>reconnects: </td><td> 0 (0.00 per sec.)</td></tr>
 <tr><td> General statistics:</td><td></td></tr>
 <tr><td><td> total time: </td><td> 600.9039s</td></tr>
 <tr><td><td>total number of events: </td><td> 811</td></tr>
 <tr><td>Latency (ms):</td><td></td></tr>
 <tr><td><td>min: </td><td> 1.55</td></tr>
<tr><td><td>avg: </td><td> 740.94</td></tr>
 <tr><td><td> max: </td><td> 7972.79 </td></tr>
 <tr><td><td>95th percentile: </td><td> 2728.81 </td></tr>
<tr><td><td>sum: </td><td> 600899.54</td></tr>
 <tr><td> Threads fairness: </td><td> </td></tr>
 <tr><td><td>events (avg/stddev): </td><td> 811.0000/0.00</td></tr>
 <tr><td><td>execution time (avg/stddev): </td><td> 600.8995/0.00</td></tr>
</table>

Select name, category, setting from  pg_settings where name in ('maintenance_work_mem','shared_buffers', 'work_mem','checkpoint_completion_target','checkpoint_timeout','min_wal_size','max_wal_size','bgwriter_lru_maxpages','bgwriter_lru_multiplier','effective_cache_size','random_page_cost','wal_compression','fsyncoff','full_page_writes','synchronous_commit');


|name|category|default|new|
|:-:|:-:|:-:|:-:|
|bgwriter_lru_maxpages|Resource Usage / Background Writer|100|1000|
|bgwriter_lru_multiplier|Resource Usage / Background Writer|2|10|
|checkpoint_completion_target|Write-Ahead Log/ Checkpoints|0.9|0.5|
|checkpoint_timeout| Write-Ahead Log / Checkpoints|300s|1h|
|effective_cache_size|Query Tuning / Planner Cost Constants|524288|20GB|
|full_page_writes|Write-Ahead Log / Settings|on|off|
|maintenance_work_mem|Resource Usage / Memory|65536|2GB|
|max_wal_size|Write-Ahead Log / Checkpoints|1024|10GB|
|min_wal_size|Write-Ahead Log / Checkpoints|80|1GB|
|random_page_cost|Query Tuning / Planner Cost Constants|4|1|
|shared_buffers| Resource Usage / Memory|16384|3GB|
|synchronous_commit|Write-Ahead Log / Settings|on|off|
|wal_compression|Write-Ahead Log / Settings|off|on|
|work_mem|Resource Usage / Memory|4096|16MB|
|(14 rows)||||


<table>
 <th>queries performed:</th>
 <tr><td><td>read:</td><td>14988</td></tr>
 <tr><td></td><td> write: </td><td>15486</td></tr>
 <tr><td></td><td> other: </td><td>2298</td></tr>
 <tr><td><td> total: </td><td>32772</td></tr>
 <tr><td>transactions: </td><td> 1148   (1.91 per sec.)</td></tr>
<tr><td><td>queries: </td><td> 32772  (54.61 per sec.)</td></tr>
 <tr><td><td>ignored errors: </td><td> 4      (0.01 per sec.)</td></tr>
 <tr><td><td>reconnects: </td><td> 0 (0.00 per sec.)</td></tr>
 <tr><td> General statistics:</td><td></td></tr>
 <tr><td><td> total time: </td><td> 600.0980s</td></tr>
 <tr><td><td>total number of events: </td><td> 1148</td></tr>
 <tr><td>Latency (ms):</td><td></td></tr>
 <tr><td><td>min: </td><td>  1.56</td></tr>
<tr><td><td>avg: </td><td> 522.73</td></tr>
 <tr><td><td> max: </td><td> 7627.10 </td></tr>
 <tr><td><td>95th percentile: </td><td> 1973.38 </td></tr>
<tr><td><td>sum: </td><td> 600092.50</td></tr>
 <tr><td> Threads fairness: </td><td> </td></tr>
 <tr><td><td>events (avg/stddev): </td><td> 1148.0000/0.00</td></tr>
 <tr><td><td>execution time (avg/stddev): </td><td> 600.0925/0.00</td></tr>
 </TABLE>
 21. Эффективность выросла до 150%, и  в приципе ошибок стало даже меньше. Латентность тоже знизилась

 Причины:
 1) Более оптимальные настройки памяти
 2) Уменьшен интервал сброса файлов на диск, больший объем крутится в операвтиной памяти
 3) Испоьзуются полшие страницы
 4) Несихронный комит
 5) Увеличины размеры WAL - файлов