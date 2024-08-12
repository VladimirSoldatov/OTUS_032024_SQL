1. Работаю в Windows. Нет прав администратора.
2. Использую VMWARE.
3. Скачал с сайта предустановленный образ с развернутой Ubuntu 23.10.
4. Создал виртуальную машину Linux без установки, подключив образ к систему в качестве HDD.
5. С помощью консоли Windows команды ssh-keygen -t ed25519 создал key pair.
6. С помощью команды type c:\users\vvsoldatov\ssh\id_rsa.pub | ssh osboxes@[имя сервера] -p [порт] "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys" перенес сертификат на сервер.
7. Проверил подключениe, работает.
8. Поставил PostgreSQL 15, используя комманду sudo apt install postgresql.
9. Отредактировал pg_hba.conf заменив настройки локального подключения для пользователя postgres как trust вместо peer.
10. Перезапустил службу sudo systemctl restart postgresql.
11. sudo -u postgres psql -- подключаемся к БД
12. Создаем 4 кластера командой sudo pg_createcluster -u postgres 15 mainX, где X от ничего до 3.
13. Команда pg_lsclustres покажет 4 кластера с IP - портами 5432, 5433, 5434, 5435 с состоянием <b>down</b>
14. Перезапустим службу Постгрес командой <b> sudo systemctl restart postgresql</b>
15. Команда pg_lsclustres покажет 4 кластера с IP - портами 5432, 5433, 5434, 5435 с состоянием <b>online</b>
16. create database test_db_replica; -- создаем на первых трех кластерах БД test_db_replica
17. Не забываем отредактировать файл pg_hba.conf <b> sudo -u postgres nano  /etc/postgresql/15/mainX/pg_hba.conf</b>
18. show wal_level; -- проверяем текущий статус настроек WAL - файлов

|wal_level|
|:-:|
|replica|

19. alter system set wal_level = logical; -- поменяем формат WAL для логической репликации для первых трех кластеров
20. \q -- выйдем из сервера
21. sudo systemctl restart postgresql -- перезапустим службу
22. show wal_level; -- проверяем текущий статус настроек WAL - файлов

|wal_level|
|:-:|
|logical|

23. create table testX(id int not null); -- создаем две таблый на трех первых кластерах в БД test_db_replica. <b> Это важный момент, а то можно в postgres все насоздавать</b> 

<center>Создаем на кластере 1</center>

24. create publication test_publication for table public.test1 - создаем публикацию на кластере порт 5432
25. \dRp+ -- проверяем корректность публиции

<center> Publication test_publication</center>

| Owner| All tables|Inserts|Updates|Deletes|Truncates|Via root|
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|postgres|f|t|t|t|t|f|

Tables:<br>
&emsp; &emsp; &emsp; "public.test1"

<center>Создаем на кластере 2</center>

26. create publication test_publication for table public.test2
27. \dRp+ -- проверяем корректность публиции

<center> Publication test_publication</center>

| Owner| All tables|Inserts|Updates|Deletes|Truncates|Via root|
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|postgres|f|t|t|t|t|f|

Tables:<br>
&emsp; &emsp; &emsp; "public.test2"

<center>Создаем на кластере 1</center>

28. create subscription test_subscription connection 'host=localhost port=5433 user=postgres dbname=test_db_replica password=postgres' publication test_publication with (copy_data = false);
NOTICE:  created replication slot "test_subscription" on publisher -- выдает ошибку

29. Снова правим файл pg_hba.conf, прописыывая авторизацию md5
30. ALTER USER postgres WITH PASSWORD 'postgres'; -- устанавливаем пароль на учетную запись

31. Повторяем шаг 28, но уже успешно

<center>Создаем на кластере 2</center>

32. create subscription test_subscription connection 'host=localhost port=5432 user=postgres dbname=test_db_replica password=postgres' publication test_publication with (copy_data = false);
NOTICE:  created replication slot "test_subscribtion" on publisher -- выдает ошибку

33. Снова правим файл pg_hba.conf, прописыывая авторизацию md5
34. ALTER USER postgres WITH PASSWORD 'postgres'; -- устанавливаем пароль на учетную запись

35. Повторяем шаг 32, но уже успешно

<center>Проверка подписок на обоих кластерах</center>

36. \dRs+

<center> List of subscriptions</center>

| Name| Owner|Enable|Publication|Binary|Streaming|Two-Phase commit|Disable on error|Synchronous commit|Conninfo|Skip LSN|
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|test_subscriptions|postgres|t|<test_puplication>|f|f|d|f|off|...|0/0|

<center>Проверяем на кластере 1</center>

37. select * from test1;

| ID|
|:-:|
|1|
|2|
|3|
|4|
|5|
|6|
|7|
|8|
|9|
|10|

38. select * from test2;

| ID|
|:-:|
|11|
|12|
|13|
|14|
|15|
|16|
|17|
|18|
|19|
|20|

<center>Проверяем на кластерах 2,3</center>

39. Аналогичная картина



40. Для создания  горячего реплицирования для высокой доступности использовался кластер 4 с портом 5435.

<center>Настройка мастера</center>

41. alter system set wal_level = hot_standby; -- выставляем уровень WAL в значение hot_standby
42. alter system set archive_mode = on; -- включаем режим архивирования
43. alter system set archive_command = 'cd .'; -- shell команда - заглушка
44. alter system set max_wal_senders = 8; -- параметр количества макимального подключения replica
45. alter system set hot_standby = on; -- включаем горячее реплицирование
46. Редактируем pg_hba.conf *-- host    replication    postgres    127.0.0.1/32    md5*
47. Если это было бы ВМ, то тут вместо 127.0.0.1/32 был бы адрес реплики

<center>Настройка реплики</center>

48. alter system set wal_level = hot_standby; -- выставляем уровень WAL в значение hot_standby
49. alter system set archive_mode = on; -- включаем режим архивирования
50. alter system set archive_command = 'cd .'; -- shell команда - заглушка
51. alter system set max_wal_senders = 8; -- параметр количества макимального подключения мастера
52. alter system set hot_standby = on; -- включаем горячее реплицирование
53. Редактируем pg_hba.conf *-- host    replication    postgres    127.0.0.1/32    md5*
54. Если это было бы ВМ, то тут вместо 127.0.0.1/32 был бы адрес мастера

55. Отключаем кластер реплики sudo pg_ctlcluster 15 main4
56. sudo - postgres rm -rf /var/lib/postgresql/15/main4; -- и удаляем все файлы в папке main4
57. sudo -u postgres mkdir /var/lib/postgresql/15/main4; -- пересоздаем папку
58. sudo chmod go-rwx /var/lib/postgresql/15/main4 -- выдаем права 750

59. Создаем пароль для учетки postgres в мастере

60. На реплике выполняем sudo -u postgres pg_basebackup -P -R -X stream -c fast -h localhost -p 5434  -U postgres -D /var/lib/postgresql/15/main4
61. Создаеется в папке реплики полная бинарная копия файлов с мастера
62. Понимаем реплику.

63. Возникшие сложности.
- следить за возможность авторизации, а это настройка pg_hba.conf и ALTER ROLE  для пользователя postgres или иной учетной записью
- права на паку с БД должны быть 750 и владелей postgres
-  логическая репликация начинается с момента создания подписки
- подписка позволяет добавлять записи минуя подписки. То есть на кластере 1 я могу выполнить INSERT в test2. Это повлияет на кластер 1 и не повлияет на 2,3,4