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
13. Остановил службу Postgres командой sudo systemctl start postgresql
14. Проверяем, что сервер SQL работает командой sudo -u postgres pg_lsclusters 

|Ver Cluster|Port| Status|Owner|Data directory |Log file|
|:-:|:-:|:-:|:-:|:-:|:-:|
|15  main| 5432 |online|postgres|/mnt/data/15/main| /var/log/postgresql/postgresql-15-main.log|
|||||||

15. Зашел в БД командой sudo -u postgres psql -d demo_otus
16. Создал таблицу -> create table test(c1 text);
<center>

|Команды|
|:-:|
|create table test(c1 text);|
|insert into test values('1');|
|\q|
|

</center>

16. Остановлен кластер - sudo systemctl stop postgresql
17. Создан новый диск к ВМ размером 10GB
18. Диск примонтирован к системе
19. https://www.digitalocean.com требует VPN

<center>

|Команды|
|:-:|
|sudo apt update|
|sudo apt install parted|
|'sudo parted -l \| grep Error|
|lsblk|
|sudo parted /dev/sdb mklabel msdos|
|sudo parted -a opt /dev/sdb mkpart primary ext4 0% 100%|
|sudo mkfs.ext4 -L datapartition /dev/sdb1|
|sudo mkdir -p /mnt/data|
|sudo mount -o defaults /dev/sdb1 /mnt/data|
|sudo systemctl stop postgresql|
|
</center>

20. Были минорные cложности на данном этапе
21. Отредактирован /etc/fstab - LABEL=datapartition /mnt/data ext4 defaults 0 2
22. sudo reboot
23. sudo chown -R postgres:postgres /mnt/data/
24. mv /var/lib/postgresql/15 /mnt/data
25. sudo -u postgres pg_ctlcluster 15 main start
26. Возникла ошибка
27. Перенастроен postgres.conf в части переменной data_directory
28. Возкла ошибка с доступом к какому-то файлу, добавлены права для пользователя postgres
29. sudo systemctl restart postgresql
30. Проверены данные на месте sudo -u postgres psql -d demo_otus и select * from test;



 