# Введние ADO.NET

Существует три разных способа применения ADO.NET:

1. Подключенный уровень - кодовая база подключается к лежащему в основе храниищу данных и отключается от него, взаимодействие с хранилищем данных посредством подключений, объектов команд и объектов чтения данных.

2. Автономный уровень - манипулированием объектов DataTable (он внутри DataSet) как копией внешних данных. А уже для отправки изменения в хранилище данных применяется адаптер данных.

3. Применение ORM, такой как Entity Framework. Применение объектов C#, предоставляющие данные в ориентированной на приложение манере. Абстрагирование от кода доступа к данным. Взаимодействие с базами данных с использованием строго типизированных запросов LINQ.

Поставщик данных - набор типов, определенных в отдельном пространстве имен, которым известно, как взаимодействовать с конкретным источником данных. Все основные классы являются производными от самого базового класса DbConnection, реализуют идентичные интерфейсы IDBConnection.

Некоторые типы поставщиков данных ADO.NET:

Тип         | Базовый класс | Интерфейсы     | Описание  
------------|---------------|----------------|------------
Connection  | DbConnection  | IDbConnection  | Отключение и подключение к хранилищу данных, доступ к связанной транзакции
Command     | DbCommand     | IDbCommand     | Запрос SQL либо хранимая процедура, доступ к объекту чтения данных поставщика
DataReader  | DbDataReader  | IDbDataReader IDbDataRecord | Доступ к данным в направлении вперед для чтения с использованием SQL курсора
DataAdapter | DbDataAdapter | IDataAdapter IDbDataAdapter | Передача объектов DataSet между кодом и хранилищем данных, содержит подключение и команды выборки, вставки, изменения и удаления данных
Parameter   | DbParameter   | IDataParameter IDbParameter | Именованный параметр параметризированного запроса
Transaction | DbTransaction | IDbTransaction | Именованный параметр параметризированного запроса

## Примитивы базы данных

Пространство имен System.Data содержит различные примитивы базы данных, реализуемые поставщиками данных.

Тип         | Описание
------------|---------------
Constraint  | Ограничение для заданного объекта DataColumn
DataColumn  | Одиночный столбец объекта DataTable
DataRelation | Отношение "родительский-дочерний" между двумя объектами DataTable
DataRow     | Одиночная строка объекта DataTable
DataSet     | Находящийся в памяти кеш данных, сотоящий из любого количества взаимосвязанных объектов DataTable
DataTable   | Табличный блок данных в памяти
DataTableReader | Трактовка объекта DataTable как курсор чтения данных в одном направлении
DataView  | Настраиваемое представление для сортировки, фильтрации, поиска, редактирования и навигации
IDataAdapter | Поведение объекта адаптера данных
IDataParameter  | Поведение объекта параметра
IDataReader  | Поведение объекта чтения данных
IDbCommand  | ПОведение команды
IDbDataAdapter | Расширяет интерфейс IDataAdapter в целях дополнительной функциональности
IDbTransaction |  Поведение объекта транзакции

>метод IsDBNull(int i) объекта DataTable позволяет определить, установлено ли указанное поле в null. 

Некторые члены типа DbConnection:

Член                     | Описание
-------------------------|-------------------------
BeginTransaction()       | Метод начинающий транзакцию базы данных
ChangeDatabase()         | Изменение базы данных, связанной с открытым подключением
ConnectionTimeout        | Свойство только для чтения вертающее промежуток времени, в течении которого ожидается установка соединения, прежде чем будет сгенерирована ошибка (стандарт - 15 секунд)
Database                 | Свойство только для чтения вертающее имя базы данных
DataSource               | Свойство только для чтения вертающее местоположение базы данных
GetSchema()              | Вертает объект DataTable с информацией по схеме источника данных
Satate                   | Свойство только для чтения вертает текущее состояние подключения (enum ConnectionState)

## Подключение к базе данных

Пример обычной строки подключения:

```csharp
string connectionString = @"data source = DESKTOP-Q5PLE8H\SQLEXPRESS; Initial Catalog = Lesson7; Integrated Security = True; Connect Timeout = 3";
```
Параметры строки получения:

- Application Name: название приложения. Может принимать в качестве значения любую строку. Значение по умолчанию: ".Net SqlClient Data Provide"

- AttachDBFileName: хранит полный путь к прикрепляемой базе данных

- Connect Timeout: временной период в секундах, через который ожидается установка подключения. Принимает одно из значений из интервала 0–32767. По умолчанию равно 15.

В качестве альтернативного названия параметра может использоваться Connection Timeout

- Data Source: название экземпляра SQL Servera, с которым будет идти взаимодействие. Это может быть название локального сервера, например, "EUGENEPC/SQLEXPRESS", либо сетевой адрес.

В качестве альтернативного названия параметра можно использовать Server, Address, Addr и NetworkAddress

- Encrypt: устанавливает шифрование SSL при подключении. Может принимать значения true, false, yes и no. По умолчанию значение false
- Initial Catalog: хранит имя базы данных

В качестве альтернативного названия параметра можно использовать Database

- Integrated Security: задает режим аутентификации. Может принимать значения true, false, yes, no и sspi. По умолчанию значение false

В качестве альтернативного названия параметра может использоваться Trusted_Connection

- Packet Size: размер сетевого пакета в байтах. Может принимать значение, которое кратно 512. По умолчанию равно 8192

- Persist Security Info: указывает, должна ли конфиденциальная информация передаваться обратно при подключении. Может принимать значения true, false, yes и no. По умолчанию значение false

- Workstation ID: указывает на рабочую станцию - имя локального компьютера, на котором запущен SQL Server

- Password: пароль пользователя

- User ID: логин пользователя

- ExecuteNonQuery() - возвращает число обработанных строк, или -1 если ни одна строка не обработана.

Альтернативно можно использовать построитель строки подключения:
```csharp
var connectionStringBuilder = new SqlConnectionStringBuilder
{
    DataSource = @"(localdb)\MSSQLLocalDB",
    InitialCatalog = "demo",
    Pooling = true,
    IntegratedSecurity = true,
};
```

## Фабрика поставщика данных

Модель фабрики поставщиков данных .NET DbProviderFactory позволяет строить единую кодовую базу на основе обобщенных типов доступа к данным. Модель фабрики поставщиков данных ADO.NET позволяет строить единествунную кодовую базу, потребляющую разные типы поставщиков данных.

Пример получения определенного поставщика данных SQL:
```csharp
DbProviderFactory sqlFactory = DbProviderFactories.GetFactory("System.Data.SqlClient");
```
Пример применения фабрики поставщика данных:
```csharp
//конфиг файл:
...
  <appSettings>
    <add key="provider" value="System.Data.SqlClient"/>
    <add key="connectionString" value="Data Source=(localdb)\MSSQLLocalDB;Initial Catalog=autolot;Integrated Security=True"/>
  </appSettings>
//приложение:
string dataProvider = ConfigurationManager.AppSettings["provider"];
string connectionString = ConfigurationManager.AppSettings["connectionString"];
DbProviderFactory factory = DbProviderFactories.GetFactory(dataProvider);
using (DbConnection connection = factory.CreateConnection())
{
    if (connection == null)
    {
        WriteLine("Ошибка создания объекта Connection");
        ReadKey();
        return;
    }
    connection.ConnectionString = connectionString;
    connection.Open();
    DbCommand command = factory.CreateCommand();
    if (command == null)
    {
        WriteLine("Ошибка создания объекта Command");
        ReadKey();
        return;
    }
    command.Connection = connection;
    command.CommandText = "SELECT * FROM Inventory";
    using (DbDataReader dataReader = command.ExecuteReader())
    {
        WriteLine("Таблица Inventory:");
        while (dataReader.Read())
            WriteLine($"-> Автомобиль {dataReader["CarId"]} это {dataReader["Make"]}.");
    }
}
ReadKey();
```

## Объект постоитель строки подключения

Позволяет устанавливать пары "имя-значение" строки подключения к базе данных с примененеием строго типизированных свойств.

Пример:
```csharp
SqlConnectionStringBuilder sqlStringBuilder = new SqlConnectionStringBuilder
{
    InitialCatalog = "autolot",
    DataSource = @"PNZASUTP\SQLEXPRESS",
    ConnectTimeout = 30,
    IntegratedSecurity = true,
};
using (SqlConnection connection = new SqlConnection())
{
    connection.ConnectionString = sqlStringBuilder.ConnectionString;
    connection.Open();
    Console.WriteLine($"Состояние соединения: {connection.State}");
}
string connectStr = @"Data Source=PNZASUTP\SQLEXPRESS;Initial Catalog=AutoLot;Integrated Security=True";
SqlConnectionStringBuilder sqlStringBuilder2 = new SqlConnectionStringBuilder(connectStr);
sqlStringBuilder2.ConnectTimeout = 5;
```

## Объект - команда

Производный от DbCommand тип является объектно-ориентированным представлением SQL-запроса, имени таблицы или хранимой процедуры. Тип устанавливается свойством CommandType - StoredProcedure, TableDirect, Text (значение по умолчанию). Еще команде обязательно нужно указать подключение к базе данных.

Некторые члены типа DbCommand:

Член                     | Описание
-------------------------|-------------------------
CommandTimeout           | Установка время ожидания, пока не завершится попытка выполнить команду и сгенерится ошибка. Стандарт - 30.
Connection               | Установка объекта DbConnection, применяемый текущим объектом-командой
Parameters               | Коллекция типов DbParameter для параметризированного запроса
Cancel()                 | Отменяет выполнение команды
ExecuteReader()          | Выполняет запрос и вертает объект поставщика данных DbDataReader, допускающий только для чтения доступ к результату запроса в одном направлении
ExecuteNonQuery()        | Выполняет оператор SQL.
ExecuteScalar()          | Легковесная верия ExecuteReader() для одноэлементных запросов
Prepare()                | Подготовленная версия команды для источника данных. Такая команда-запрос будет выполняться быстрее, особенно для команды, выполняемой многократно с разными параметрами.

## Чтение данных

Самый быстрый и простой способ получения информации из базы данных - тип DBDataReader. Полезен когда базе данных отправляются только запросы с операторами выборки данных. Очень полезен, когда нужно быстрой пройтись по большому объему в базе без необходимости хранить в памяти. Важной особенностью является то, что объект чтения данных удерживает подключения к источнику данных открырым до тех пор, пока оно явным образом не будет закрыто.

Пример простого чтения данных:
```csharp
...
using (SqlConnection connection = new SqlConnection())
{
    connection.ConnectionString = sqlStringBuilder.ConnectionString;
    connection.Open();
    Console.WriteLine($"Состояние соединения: {connection.State}");
    SqlCommand command = new SqlCommand("SELECT * FROM Inventory", connection);
    using (SqlDataReader reader = command.ExecuteReader())
    {
        while (reader.Read())
        {
            Console.WriteLine($"-> Марка: {reader["Make"]} Цвет: {reader["Color"]}");
        }
    }
}
```

## Добавление, изменение и удаление данных

Инфраструктура доступа к данным полностью поддерживает функциональность создания, чтения, обновления и удаления (create, read, update, delete - CRUD).

Можно работать с базой данных напрямую из своей библиотеки, а можно использовать автономный уровень ADO.NET, последнее более удобно.



## Транзакции базы данных

Помещенные команды базы данных внутрь транзакции гарантирует, что все взаимосвязанные команды будут выполнены как единое целое. Если любая часть транзакции откажет, то будет произведен откат всей операции в исходное состояние. Если команды завершены успено, то производится фиксация транзакции.

Метод Commin() вызывается, если все операции в базе данных завершились успешно, а ROllback() наоборот, информирует СУБД о необходимости проигнорировать все изменения и оставить первоначальные данные нетронутыми.

Пример применения транзакции базы данных:
```csharp

```

## Отслеживание изменения состояния соеднинения

В объекте соединения существует событие, позволяющее отслеживать изменение состояния соеднения. Пример:
```csharp
            connection.StateChange += Connection_StateChange;
            connection.Open();
...
private static void Connection_StateChange(object sender, System.Data.StateChangeEventArgs e)
{
    var s = sender as SqlConnection;
    Console.WriteLine($"Сбор статистики: {s.StatisticsEnabled} имя источника: {s.DataSource} новое состояние: {e.CurrentState} текущее состояние: {e.OriginalState}");
}
```

