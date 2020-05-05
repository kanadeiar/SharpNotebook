# Основы WPF Базы данных

## Получение данных из базы данных

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

ExecuteReader() - получение результата SQL-запроса или хранимой процедуры, возвращает полученное значение типа System.Data.SqlClient.SqlDataReader, допускающее построчное чтение результата.

Пример чтения результата из таблицы базы данных:
```csharp
string sql = "SELECT * FROM People";
public MainWindow()
{
    InitializeComponent();
    string connectionString = @"data source = DESKTOP-Q5PLE8H\SQLEXPRESS; Initial Catalog = Lesson7; Integrated Security = True";
    using (SqlConnection connection = new SqlConnection(connectionString))
    {
        connection.Open();
        SqlCommand command = new SqlCommand(sql, connection);
        SqlDataReader reader = command.ExecuteReader(CommandBehavior.CloseConnection);
        if (reader.HasRows)
        {
            while (reader.Read())
            {
                var id = Convert.ToInt32(reader.GetValue(0));
                var fio = reader.GetString(1);
                var birthday = reader.GetString(2);
                var email = reader["Email"];
                var phone = reader.GetString(reader.GetOrdinal("Phone"));
                MessageBox.Show($"{id}\n{fio} {birthday}\n{email} {phone}");
            }
        }
        reader.Close();
        connection.Close();
    }
}
```
AddWithValue() - добавление параметра со значением.

Пример чтения с параметром из таблицы базы данных:
```csharp
string sqlWhere = "SELECT * FROM People WHERE Birthday = @Birthday";
public MainWindow()
{
    InitializeComponent();
    string connectionString = @"data source = DESKTOP-Q5PLE8H\SQLEXPRESS; Initial Catalog = Lesson7; Integrated Security = True";
    using (SqlConnection connection = new SqlConnection(connectionString))
    {
        connection.Open();
        SqlCommand command = new SqlCommand(sqlWhere, connection);
        command.Parameters.AddWithValue("@Birthday","03.03.2011");
        SqlDataReader reader = command.ExecuteReader(CommandBehavior.CloseConnection);
        if (reader.HasRows)
        {
            while (reader.Read())
            {
                var id = Convert.ToInt32(reader.GetValue(0));
                var fio = reader.GetString(1);
                var birthday = reader.GetString(2);
                var email = reader["Email"];
                var phone = reader.GetString(reader.GetOrdinal("Phone"));
                MessageBox.Show($"{id}\n{fio} {birthday}\n{email} {phone}");
            }
        }
        reader.Close();
        connection.Close();
    }
}
```

## Адаптер соединения с базой данных

Передача данных из БД в DataSet и DataTable - метод Fill(). Сохранение изменений в БД из DataAdapter - метод Update().

SelectCommand - управляет потоком данных из БД в DataTable или DataSet;

Пример заполения таблицы данными из БД:
```csharp
string sql = "SELECT * FROM People";
public MainWindow()
{
    InitializeComponent();
    string connectionString = @"data source = DESKTOP-Q5PLE8H\SQLEXPRESS; Initial Catalog = Lesson7; Integrated Security = True";
    using (SqlConnection connection = new SqlConnection(connectionString))
    {
        SqlDataAdapter adapter = new SqlDataAdapter();
        adapter.SelectCommand = new SqlCommand(sql, connection);
        DataTable dtable = new DataTable();
        adapter.Fill(dtable);
        foreach (DataRow el in dt.Rows)
            MessageBox.Show($"{el["FIO"]} {el["Birthday"]} {el["Email"]}");
    }
}
```
Пример заполенения источника данных:
```csharp
string sql = "SELECT * FROM People";
public MainWindow()
{
    InitializeComponent();
    string connectionString = @"data source = DESKTOP-Q5PLE8H\SQLEXPRESS; Initial Catalog = Lesson7; Integrated Security = True";
    using (SqlConnection connection = new SqlConnection(connectionString))
    {
        SqlDataAdapter adapter = new SqlDataAdapter();
        adapter.SelectCommand = new SqlCommand(sql, connection);
        DataSet ds = new DataSet();
        adapter.Fill(ds);
        foreach (DataRow el in ds.Tables[0].Rows)
            MessageBox.Show($"{el["FIO"]} {el["Birthday"]} {el["Email"]}");
    }
}
```
Пример использования адаптера с вручную написанными запросами:
```csharp
public MainWindow()
{
    InitializeComponent();
    string connectionString = @"data source = DESKTOP-Q5PLE8H\SQLEXPRESS; Initial Catalog = Lesson7; Integrated Security = True";
    using (SqlConnection connection = new SqlConnection(connectionString))
    {
        SqlDataAdapter adapter = new SqlDataAdapter();
        SqlCommand selectCmd = new SqlCommand("SELECT * FROM People", connection);
        adapter.SelectCommand = selectCmd;
        SqlCommand insertCmd = new SqlCommand(@"INSERT INTO People (FIO,Birthday,Email,Phone) VALUES (@FIO,@Birthday,@Email,@Phone); SET @ID = @@IDENTITY;", connection);
        insertCmd.Parameters.Add("@FIO", SqlDbType.VarChar,256,"FIO");
        insertCmd.Parameters.Add("@Birthday", SqlDbType.VarChar,256,"Birthday");
        insertCmd.Parameters.Add("@Email", SqlDbType.VarChar,100,"Email");
        insertCmd.Parameters.Add("@Phone", SqlDbType.VarChar,100,"Phone");
        SqlParameter insertParam = insertCmd.Parameters.Add("@ID",SqlDbType.Int,0,"ID");
        insertParam.Direction = ParameterDirection.Output;
        adapter.InsertCommand = insertCmd;
        SqlCommand updateCmd = new SqlCommand(@"UPDATE People SET FIO = @FIO,Birthday = @Birthday,Email = @Email,Phone = @Phone WHERE ID = @ID", connection);
        updateCmd.Parameters.Add("@FIO", SqlDbType.VarChar,256,"FIO");
        updateCmd.Parameters.Add("@Birthday", SqlDbType.VarChar,256,"Birthday");
        updateCmd.Parameters.Add("@Email", SqlDbType.VarChar,100,"Email");
        updateCmd.Parameters.Add("@Phone", SqlDbType.VarChar,100,"Phone");
        SqlParameter updateParam = updateCmd.Parameters.Add("@ID",SqlDbType.Int,0,"ID");
        updateParam.SourceVersion = DataRowVersion.Original;
        adapter.UpdateCommand = updateCmd;
        SqlCommand deleteCmd = new SqlCommand("DELETE FROM People WHERE ID = @ID", connection);
        SqlParameter deleteParam = deleteCmd.Parameters.Add("@ID",SqlDbType.Int,0,"ID");
        deleteParam.SourceVersion = DataRowVersion.Original;
        DataSet ds = new DataSet();
        adapter.Fill(ds);
        StringBuilder sb = new StringBuilder();
        foreach (DataRow el in ds.Tables[0].Rows)
            sb.AppendLine($"{el["FIO"]} {el["Birthday"]} {el["Email"]} {el["Phone"]}");
        MessageBox.Show(sb.ToString());
    }
}
```









