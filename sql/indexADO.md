# ADO.NET

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

