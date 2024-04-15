1. Использую VMWARE.
2. Скачал с сайта предустановленный образ с развернутой Ubuntu 23.10.
3. Создал виртуальную машину Linux без установки, подключив образ к систему в качестве HDD.
4. С помощью консоли Windows команды ssh-keygen -t ed25519 создал key pair.
5. С помощью команды type c:\users\vvsoldatov\ssh\id_rsa.pub | ssh osboxes@[имя сервера] -p [порт] "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys" перенес сертификат на сервер.
6. Проверил подключени, работает.
7. Подгтовка к установке docker:
    1. Создал папку  для ключей: sudo mkdir -p /etc/apt/keyrings
    2.  Скачиваем ключи в созданную папку: curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    3. Выдаем права yf чтение всем: sudo chmod a+r /etc/apt/keyrings/docker.gpg
    4. Обновляем данные для установщика: sudo apt update
8. Установка Docker: sudo apt install docker-ce
9. Создал папку mkdir /var/lib/postgres
9. Установка контейнера из сети: sudo docker run -d  --name my-postgres -p 5432:5433 -v ~/apps/postgres:/var/lib/postgresql/data -e POSTGRES_PASSWORD=postgres -e POSTGRES_USER=postgres -e POSTGRES_DB=demo_otus postgres:15-alpine
10. Порт изменен на 5433 так как на Linux eуже установлен Postgres
11. Подключаемся к базе контейнера: psql -h localhost --port=5433 --username=postgres -d demo_otus
12. Создаем таблицу: create table persons(id int not null generated always as identity, name varchar(50));
13. Заполняем данными: 
demo_otus=# insert into persons (name) values('Nick');
INSERT 0 1
demo_otus=# insert into persons (name) values('Anna');
INSERT 0 1
14. Выводим данные: select * from persons;

|id|name|
|:-:|:-:|
|1|Nick|
|2|Anna|

14. При попытке удалить контейнер:<br>
 sudo docker rm my-postgres <br>
 ввыдает ошибку Error response from daemon:<br>
 cannot remove container "/my-postgres": container is running: stop the container before removing or force remove
15. Останавливаем контейнер: sudo docker stop my-postgre
16. Удаляем контейнер:  sudo docker rm my-postgre
17. В виду того, что конйнейнер создался под логином root, лучше выдать пользователю права на файл: sudo setfacl --modify user:<osboxes>:rw /var/run/docker.sock
18. А также включить пользователя в группу докер: <br>
sudo usermod -aG docker osboxes
19. Заново подключил командой<br> 
docker run my-postgres
20. Данные на месте