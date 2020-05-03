# Основы WPF Базы данных

## Строка подключения к базе данных

Пример обычной строки подключения:

```csharp
string connectionString = @"data source = DESKTOP-Q5PLE8H\SQLEXPRESS; Initial Catalog = Lesson7; Integrated Security = True";
```

ExecuteNonQuery() - возвращает число обработанных строк, или -1 если ни одна строка не обработана.

Пример создания базы данных и заполнения тестовыми данными:
```csharp
string create = @"CREATE TABLE [People] ([Id] INT IDENTITY(1,1) NOT NULL, [FIO] VARCHAR(256) NOT NULL, [Birthday] VARCHAR(256) NULL, [Email] VARCHAR(100) NULL, [Phone] VARCHAR(100) NULL, CONSTRAINT [PK_Id_People] PRIMARY KEY (Id));";
string insert1 = @"INSERT INTO [People] (FIO,Birthday,Email,Phone) VALUES ('Иванов Иван Иванович','01.01.2001','some@some.ru','89297771111');";
public MainWindow()
{
    InitializeComponent();
    string connectionString = @"data source = DESKTOP-Q5PLE8H\SQLEXPRESS; Initial Catalog = Lesson7; Integrated Security = True";
    using (SqlConnection connection = new SqlConnection(connectionString))
    {
        connection.Open();
        SqlCommand command1 = new SqlCommand(create, connection);
        int num = command1.ExecuteNonQuery();
        SqlCommand command2 = new SqlCommand(insert1, connection);
        int num1 = command2.ExecuteNonQuery();
        MessageBox.Show($"Результат: создание: {num} вставка: {num1}");
        connection.Close();
    }
}
```

ExecuteScalar() - возвращает единственное значение, сформированное запросом. Возвращает значение типа System.Object, требует явного приведения к требуемому типу.

Пример получения количества записей:
```csharp
string count = "SELECT COUNT(*) FROM People";
public MainWindow()
{
    InitializeComponent();
    string connectionString = @"data source = DESKTOP-Q5PLE8H\SQLEXPRESS; Initial Catalog = Lesson7; Integrated Security = True";
    using (SqlConnection connection = new SqlConnection(connectionString))
    {
        connection.Open();
        SqlCommand command = new SqlCommand(count, connection);
        int num = Convert.ToInt32(command.ExecuteScalar());
        MessageBox.Show($"Количество записей: {num}");
        connection.Close();
    }
}
```
Пример добавления новой записи:
```csharp
string count = "INSERT INTO [People] (FIO,Birthday,Email,Phone) VALUES ('Иванов Иван Иванович','01.01.2001','some@some.ru','89297771111');";
public MainWindow()
{
    InitializeComponent();
    string connectionString = @"data source = DESKTOP-Q5PLE8H\SQLEXPRESS; Initial Catalog = Lesson7; Integrated Security = True";
    using (SqlConnection connection = new SqlConnection(connectionString))
    {
        connection.Open();
        SqlCommand command = new SqlCommand(count, connection);
        int newId = Convert.ToInt32(command.ExecuteScalar());
        MessageBox.Show($"Добавлена запись с ид: {newId}");
        connection.Close();
    }
}
```



