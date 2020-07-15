# Подключенный уровень ADO.NET

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

## Объект построитель строки подключения

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
public void ProcessCreditRisk(int custId)
{
    OpenConnection();
    string fName;
    string lName;
    var cmdSelect = new SqlCommand($"SELECT * FROM Customers WHERE CustId = @id", _sqlConnection);
    cmdSelect.Parameters.Add("@id", SqlDbType.Int, -1).Value = custId;
    using (var reader = cmdSelect.ExecuteReader())
    {
        if (reader.HasRows)
        {
            reader.Read();
            fName = (string) reader["FirstName"];
            lName = (string) reader["LastName"];
        }
        else
        {
            CloseConnection();
            return;
        }
    }
    var cmdRemove = new SqlCommand($"DELETE FROM Customers WHERE CustId = @id", _sqlConnection);
    cmdRemove.Parameters.Add("@id", SqlDbType.Int, -1).Value = custId;
    var cmdInsert = new SqlCommand($"INSERT INTO CreditRisks (FirstName,LastName) VALUES (@fname, @lname)", _sqlConnection);
    cmdInsert.Parameters.Add("@fname", SqlDbType.NVarChar, -1).Value = fName;
    cmdInsert.Parameters.Add("@lname", SqlDbType.NVarChar, -1).Value = lName;
    SqlTransaction tx = null;
    try
    {
        tx = _sqlConnection.BeginTransaction(); //сама транзакция
        cmdInsert.Transaction = tx;
        cmdRemove.Transaction = tx;
        cmdInsert.ExecuteNonQuery();
        cmdRemove.ExecuteNonQuery();
        tx.Commit(); //фиксация обоих шагов транзакции
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.Message);
        tx?.Rollback(); //откат транзакции
    }
    finally
    {
        CloseConnection();
    }
}
//применение метода:
inventory.ProcessCreditRisk(1);
```

## Массовое копирование

Специально для загружки множества данных в ADO.NET предусмотрен класс SqlBulkCopy. Класс имеет метод WriteToServer(), обрабатывающий список записей более эффективно, чем insert. Может принимать в параметрах объект DataTable, объект DataReader, массив DataRow.

Пример применения массового копирования:
```csharp
//интерфейс класса чтения данных
interface IMyDataReader<T> : IDataReader
{
    List<T> Records { get; set; }
}
/// <summary> Класс чтения данных </summary>
public class MyDataReader<T> : IMyDataReader<T>
{
    private int _currentIndex = -1;
    public bool Read()
    {
        if ((_currentIndex + 1) >= Records.Count)
            return false;
        _currentIndex++;
        return true;
    }
    private readonly PropertyInfo[] _propertyInfos;
    private readonly Dictionary<string, int> _nameDictionary;
    public MyDataReader()
    {
        _propertyInfos = typeof(T).GetProperties();
        _nameDictionary = _propertyInfos
            .Select((x, index) => new {x.Name, index})
            .ToDictionary(p => p.Name, p => p.index);
    }
    public int FieldCount => _propertyInfos.Length;
    public string GetName(int i)
    {
        return i >= 0 && i < FieldCount ? _propertyInfos[i].Name : string.Empty;
    }
    public int GetOrdinal(string name)
    {
        return _nameDictionary.ContainsKey(name) ? _nameDictionary[name] : -1;
    }
    public object GetValue(int i)
    {
        return _propertyInfos[i].GetValue(Records[_currentIndex]);
    }
...
//класс массового копирования
public static class ProcessBulkCopy
{
    private static readonly string _connectionString =
        @"Data Source=(localdb)\MSSQLLocalDB;Initial Catalog=AutoLot;Integrated Security=True";
    private static SqlConnection _sqlConnection = null;
    private static void OpenConnection()
    {
        _sqlConnection = new SqlConnection { ConnectionString = _connectionString };
        _sqlConnection.Open();
    }
    private static void CloseConnection()
    {
        if (_sqlConnection?.State != ConnectionState.Closed)
            _sqlConnection.Close();
    }
    public static void ExecuteBulkCopy<T>(IEnumerable<T> records, string tableName)
    {
        OpenConnection();
        using (SqlConnection conn = _sqlConnection)
        {
            SqlBulkCopy bc = new SqlBulkCopy(conn)
            {
                DestinationTableName = tableName
            };
            var reader = new MyDataReader<T> {Records = records.ToList()};
            try
            {
                bc.WriteToServer(reader);
            }
            catch (Exception ex)
            {
                Console.WriteLine("Ошибка!\n" + ex.Message);
            }
            finally
            {
                CloseConnection();
            }
        }
    }
}
//использование
var cars = new List<Car>()
{
    new Car {Make = "Honda", Color = "Black", Name = "One"},
    new Car {Make = "Volvo", Color = "Yellow", Name = "Lang"},
    new Car {Make = "VW", Color = "White", Name = "Three"},
    new Car {Make = "Toyota", Color = "Green", Name = "Five"},
};
ProcessBulkCopy.ExecuteBulkCopy(cars, "Inventory");
Console.WriteLine("После добавления:");
list = inventory.GetAllInventory(); 
Console.WriteLine("Ид\tМарка\tЦвет\tНазвание");
foreach(var el in list)
    Console.WriteLine($"{el.CarId}\t{el.Make}\t{el.Color}\t{el.Name}");
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









