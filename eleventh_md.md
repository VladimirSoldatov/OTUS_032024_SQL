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

|deadlock_timeout|
|:-:|
|200s|

