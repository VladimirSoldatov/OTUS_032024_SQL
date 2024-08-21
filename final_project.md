1. Создаем VM на Windows 10.
2. Скачиваем последний дистрибутив MS SQL Express 2019
3. Скачиваем [https://www.sqlskills.com/resources/conferences/salesdb2014.zip] бэкап базы
4. Скачиваем SQL Server Management Studio (SSMS) последней версии
5. Разворачиваем бэкап
6. Запрос данных на структуру
<b>SELECT<br>
 t.TABLE_SCHEMA<br>
, t.table_name as "Таблица"<br>
, t.TABLE_TYPE<br>
  , c.ordinal_position as "№ п.п",<br>
  c.column_name as "Поле",<br>
  c.DATA_TYPE as "Тип",<br>
  c.is_nullable as "NULL"<br>
  , isnull(string_agg(kcu.CONSTRAINT_NAME,','),'') as  Constraint_key<br>
FROM<br>
  information_schema.tables t<br>
 left  join information_schema.columns c on c.TABLE_NAME = t.TABLE_NAME<br>
	AND c.table_schema = t.table_schema <br>
	left JOIN INFORMATION_SCHEMA.TABLE_CONSTRAINTS tc on tc.TABLE_NAME = c.TABLE_NAME and t.TABLE_SCHEMA = tc.TABLE_SCHEMA <br>
	left join INFORMATION_SCHEMA.KEY_COLUMN_USAGE kcu <br>
	  ON tc.TABLE_SCHEMA = kcu.TABLE_SCHEMA and kcu.COLUMN_NAME = c.COLUMN_NAME <br>
  AND tc.TABLE_NAME = kcu.TABLE_NAME <br>
  AND tc.CONSTRAINT_NAME = kcu.CONSTRAINT_NAME <br>
  group by   t.TABLE_SCHEMA,t.TABLE_TYPE, t.TABLE_NAME, c.ORDINAL_POSITION, c.COLUMN_NAME, c.DATA_TYPE, c.IS_NULLABLE<br></b>
7. Результат запроса на структуру
<table>
<thead>
<th>  TABLE_SCHEMA</th><th>Таблица </th><th> TABLE_TYPE	</th><th>№ п.п	</th><th> Поле	</th><th>Тип</th><th>	NULL	</th><th> Constraint_key</th><thead>
<tbody>
<tr><td>dbo</td><td>Customers</td><td>BASE TABLE</td><td>1</td><td> CustomerID</td><td>int</td><td>NO</td><td>CustomerPK</tr>
<tr><td>dbo</td><td>Customers</td><td>BASE TABLE</td><td>2</td><td>FirstName</td><td>nvarchar</td><td>NO</td><td></td><td></tr>
<tr><td>dbo</td><td>Customers</td><td>BASE TABLE</td><td>3</td><td>MiddleInitial</td><td>nvarchar</td><td>YES</td><td></td><td></tr>	
<tr><td>dbo</td><td>Customers</td><td>BASE TABLE</td><td>4</td><td>LastName</td><td>nvarchar</td><td>NO</td><td></td><td></tr>
<tr><td>dbo</td><td>Employees</td><td>BASE TABLE</td><td>1</td><td>EmployeeID</td><td>int</td><td>NO</td><td>EmployeePK
<tr><td>dbo</td><td>Employees</td><td>BASE TABLE</td><td>2</td><td>FirstName</td><td>nvarchar</td><td>NO</td><td></td><td></tr>	
<tr><td>dbo</td><td>Employees</td><td>BASE TABLE</td><td>3</td><td>MiddleInitial</td><td>nvarchar</td><td>YES</td><td></td><td></tr>	
<tr><td>dbo</td><td>Employees</td><td>BASE TABLE</td><td>4</td><td>LastName</td><td>nvarchar</td><td>NO</td><td></td><td></tr>	
<tr><td>dbo</td><td>Products</td><td>BASE TABLE</td><td>1</td><td>ProductID</td><td>int</td><td>NO</td><td>ProductsPK</tr>
<tr><td>dbo</td><td>Products</td><td>BASE TABLE</td><td>2</td><td>Name</td><td>nvarchar</td><td>NO</td><td></td><td></tr>	
<tr><td>dbo</td><td>Products</td><td>BASE TABLE</td><td>3</td><td>Price</td><td>money</td><td>YES</td><td></td><td></tr>	
<tr><td>dbo</td><td>Sales</td><td>BASE TABLE</td><td>1</td><td>SalesID</td><td>int</td><td>NO</td><td>SalesPK</tr>
<tr><td>dbo</td><td>Sales</td><td>BASE TABLE</td><td>2</td><td>SalesPersonID</td><td>int</td><td>NO</td><td>SalesEmployeesFK</tr>
<tr><td>dbo</td><td>Sales</td><td>BASE TABLE</td><td>3</td><td>CustomerID</td><td>int</td><td>NO</td><td>SalesCustomersFK</tr>
<tr><td>dbo</td><td>Sales</td><td>BASE TABLE</td><td>4</td><td>ProductID</td><td>int</td><td>NO</td><td>alesProductsFK</tr>
<tr><td>dbo</td><td>Sales</td><td>BASE TABLE</td><td>5</td><td>Quantity</td><td>int</td><td>NO</td><td></td><td></tr>
<tr><td>dbo</td><td>sysdiagrams</td><td>BASE TABLE</td><td>1</td><td>name</td><td>nvarchar</td><td>NO</td><td>UK_principal_name</tr>
<tr><td>dbo</td><td>sysdiagrams</td><td>BASE TABLE</td><td>2</td><td>principal_id</td><td>int</td><td>NO</td><td>UK_principal_name</tr>
<tr><td>dbo</td><td>sysdiagrams</td><td>BASE TABLE</td><td>3</td><td>diagram_id</td><td>int</td><td>NO</td><td>PK__sysdiagr__C2B05B6149718787</tr>
<tr><td>dbo</td><td>sysdiagrams</td><td>BASE TABLE</td><td>4</td><td>version</td><td>int</td><td>YES</td><td></td><td></tr>
<tr><td>dbo</td><td>sysdiagrams</td><td>BASE TABLE</td><td>5</td><td>definition</td><td>varbinary</td><td>YES</td><td></td><td></tr>
</tbody></table>	
8. Запрос данных на структуру

DECLARE       @object_name SYSNAME     , @object_id INT     , @SQL NVARCHAR(MAX) 
SELECT       @object_name = '' + OBJECT_SCHEMA_NAME(o.[object_id]) + '.' + OBJECT_NAME([object_id]) +''     , @object_id = [object_id] FROM (SELECT[object_id] = OBJECT_ID('dbo.Customers', 'U')) o   

SELECT @SQL = 'CREATE TABLE ' + @object_name + CHAR(13) + '(' + CHAR(13) + STUFF((  SELECT CHAR(13) + '    , ' + c.name + ' ' +       CASE WHEN c.is_computed = 1           
THEN 'AS ' + OBJECT_DEFINITION(c.[object_id], c.column_id)
ELSE               CASE WHEN c.system_type_id != c.user_type_id                   
THEN '' + SCHEMA_NAME(tp.[schema_id]) + '.' + tp.name + ''                   
ELSE '' + UPPER(tp.name) + ''               END +               
CASE
	WHEN tp.name IN('varchar', 'char', 'varbinary', 'binary') THEN '(' + CASE WHEN c.max_length = -1                                       
		THEN 'MAX'
	ELSE CAST(c.max_length AS VARCHAR(5)) END + ')'                   
	WHEN tp.name IN('nvarchar', 'nchar')
	THEN '(' + 
	CASE WHEN c.max_length = -1                                       
	THEN 'MAX'                     
	ELSE CAST(c.max_length / 2 AS VARCHAR(5)) END + ')'                   
WHEN tp.name IN('datetime2', 'time2', 'datetimeoffset') THEN '(' + CAST(c.scale AS VARCHAR(5)) + ')'
WHEN tp.name = 'decimal' 
THEN '(' + CAST(c.[precision] AS VARCHAR(5)) + ',' + CAST(c.scale AS VARCHAR(5)) + ')' 
ELSE ''  END + '' END   
FROM sys.columns c WITH(NOLOCK)   JOIN sys.types tp WITH(NOLOCK) ON c.user_type_id = tp.user_type_id   
LEFT JOIN sys.check_constraints cc WITH(NOLOCK) ON c.[object_id] = cc.parent_object_id AND cc.parent_column_id = c.column_id  
WHERE c.[object_id] = @object_id   ORDER BY c.column_id   FOR XML PATH(''), TYPE).value('.', 'NVARCHAR(MAX)'), 1, 7, '') +')' 

select @SQL

9. Текст запроса на создание таблицы

CREATE TABLE dbo.Customers <br>(<br>CustomerID INT<br>, FirstName NVARCHAR(40)<br>, MiddleInitial NVARCHAR(40)<br>, LastName NVARCHAR(40)<br>)

### Install FreeTDS and build dependencies

The TDS foreign data wrapper requires a library that implements the DB-Library interface,
such as [FreeTDS](http://www.freetds.org).

```bash
sudo apt-get update
sudo apt-get install libsybdb5 freetds-dev freetds-common
```

Some other dependencies are also needed to install PostgreSQL and then compile tds_fdw:

```bash
sudo apt-get install gnupg gcc make
```


```bash
export TDS_FDW_VERSION="2.0.3"
sudo apt-get install wget
wget https://github.com/tds-fdw/tds_fdw/archive/v${TDS_FDW_VERSION}.tar.gz
tar -xvzf v${TDS_FDW_VERSION}.tar.gz
cd tds_fdw-${TDS_FDW_VERSION}/
make USE_PGXS=1
sudo make USE_PGXS=1 install
```
Выполнив данные действия, можно приступать 