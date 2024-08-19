1. Работаю в Windows. Нет прав администратора.
2. Использую VMWARE.
3. Скачал с сайта предустановленный образ с развернутой Ubuntu 20.04.6 LTS
4. Создал виртуальную машину Linux без установки, подключив образ к систему в качестве HDD.
5. С помощью консоли Windows команды ssh-keygen -t rsa создал key pair.
6. С помощью команды type c:\users\[имя_пользователя]\ssh\id_rsa.pub | ssh osboxes@[имя сервера] -p [порт] "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys" перенес сертификат на сервер.
7. Проверил подключени, работает.
8. Поставил PostgreSQL 16, используя комманду sudo apt install postgresql-16, после того как добавил репозитарий и публичный ключ.
9. Отредактировал pg_hba.conf заменив настройки локального подключения для пользователя postgres как trust вместо peer.
10. Перезапустил службу sudo systemctl restart postgresql.
11. Подключился как пользователь postgres - sudo psql postgres -U postgres и создал тестовую базу demo_otus.\
12. Переподключился на тестовую базу - \c demo_otus.
<center>
<table>
<tr><td>sudo -u postgres pg_lsclusters</td></tr>
<tr><td>Ver</td><td>Cluster Port </td><td>Status Owner</td><td>Data directory</td><td>Log file</td></tr>
<tr><td>16</td><td>main</td><td>5432</td><td>online</td><td>postgres</td><td>/var/lib/postgresql/16/main /var/log/postgresql/postgresql-16-main.log</td></tr>
</table>
</center>


CREATE OR REPLACE FUNCTION pract_functions.goods_report()<br>
RETURNS trigger<br>
AS<br>
$ft$<br>
BEGIN<br>

	TRUNCATE TABLE good_sum_mart;

	INSERT INTO good_sum_mart
	SELECT G.good_name, sum(G.good_price * S.sales_qty)
	FROM goods G
	INNER JOIN sales S ON S.good_id = G.goods_id
	GROUP BY G.good_name;

	RETURN NULL;
END<br>
$ft$<br>
	LANGUAGE plpgsql<br>
	SECURITY DEFINER;<br>

CREATE TRIGGER update_report_after_insert<br>
AFTER INSERT<br>
ON pract_functions.sales<br>
FOR EACH ROW<br>
EXECUTE PROCEDURE pract_functions.goods_report();<br>

CREATE TRIGGER update_report_after_update<br>
AFTER UPDATE<br>
ON pract_functions.sales<br>
FOR EACH ROW<br>
EXECUTE PROCEDURE pract_functions.goods_report();<br>

CREATE TRIGGER update_report_after_delete<br>
AFTER DELETE<br>
ON pract_functions.sales<br>
FOR EACH ROW<br>
EXECUTE PROCEDURE pract_functions.goods_report();<br>

В этом решении проблема указанная в вопросе не решается. Отчет формируется на основе текущих цен. В перспективе можно это поправить, но потребуется усложнить логику и переазаписывать только данные на день продаж. Длф этого потребуется группировка, например, по дате (день).
