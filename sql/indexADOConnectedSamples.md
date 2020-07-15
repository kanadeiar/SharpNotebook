# ADO.NET Подключенный уровень Примеры

## Чтение данных

Простой пример:
```csharp
using (SqlConnection connection = new SqlConnection())
{
    connection.ConnectionString = @"Data Source=(localdb)\MSSQLLocalDB;Initial Catalog=autolot;Integrated Security=True";
    connection.Open();
    string sql = "SELECT * FROM Inventory";
    SqlCommand command = new SqlCommand(sql, connection);
    using (SqlDataReader reader = command.ExecuteReader())
    {
        int make = reader.GetOrdinal("Make");
        int name = reader.GetOrdinal("Name");
        int color = reader.GetOrdinal("Color");
        while (reader.Read())
        {
            WriteLine($"-> Марка: {reader[make]} Название: {reader[name]} Цвет: {reader[color]}");
        }
    }
}
```
Пример чтения данных - всех столбцов таблицы:
```csharp
using (SqlConnection connection = new SqlConnection())
{
    connection.ConnectionString = sqlStringBuilder.ConnectionString;
    connection.Open();
    SqlCommand command = new SqlCommand("SELECT * FROM Inventory", connection);
    using (SqlDataReader reader = command.ExecuteReader())
    {
        while (reader.Read())
        {
            Console.WriteLine("Запись:");
            for (int i = 0; i < reader.FieldCount; i++)
                Console.WriteLine($"{reader.GetName(i)} = {reader.GetValue(i)}");
            Console.WriteLine();
        }
    }
}
```

## Простое приложение работы с базой данных

Код SQL генерации тестовой базы данных:
```csharp
--Создание таблиц с удаление уже имеющихся
IF OBJECT_ID('Person' ) IS NOT NULL DROP TABLE Person;
CREATE TABLE Person
(
id INT IDENTITY(1,1) NOT NULL,
fam NVARCHAR(256) NOT NULL,
name NVARCHAR(256) NOT NULL,
CONSTRAINT PK_id_Employee PRIMARY KEY (id),
);
--Тестовые данные
DECLARE @count_persons INT = 1000;
DECLARE @i INT = 1;
WHILE @i <= @count_persons
BEGIN
	INSERT INTO Person(fam, name)
	VALUES (CONCAT(N'Тестов', @i),CONCAT(N'Тест', @i));
	SET @i = @i + 1;
END
--Сформированные тестовые данные
SELECT id,fam,name FROM Person;
```

Пример приложения работы с базой данных:
```csharp
//главное окно
<Grid.RowDefinitions>
    <RowDefinition Height="5*"/>
    <RowDefinition/>
</Grid.RowDefinitions>
<DataGrid x:Name="DataGridDataBase" AutoGenerateColumns="False" EnableRowVirtualization="True" IsReadOnly="True" 
          ItemsSource="{Binding}">
    <DataGrid.Columns>
        <DataGridTextColumn Binding="{Binding Path=id}" Header="Id" Width="*"/>
        <DataGridTextColumn Binding="{Binding Path=fam}" Header="Фамилия" Width="*"/>
        <DataGridTextColumn Binding="{Binding Path=name}" Header="Имя" Width="*"/>
    </DataGrid.Columns>
</DataGrid>
<WrapPanel Grid.Row="1">
    <Button Content="Добавить" Margin="5" Click="ButtonAdd_OnClick"/>
    <Button Content="Редактировать" Margin="5" Click="ButtonEdit_OnClick"/>
    <Button Content="Удалить" Margin="5" Click="ButtonDelete_OnClick"/>
</WrapPanel>
private IDbConnection _connection;
private IDbDataAdapter _adapter;
private DataSet _dataSet;
public MainWindow()
{
    InitializeComponent();
}
private void MainWindow_OnLoaded(object sender, RoutedEventArgs e)
{
    var connectionStringBuilder = new SqlConnectionStringBuilder
    {
        DataSource = @"(localdb)\MSSQLLocalDB",
        InitialCatalog = "sample",
    };
    _connection = new SqlConnection(connectionStringBuilder.ConnectionString);
    _adapter = new SqlDataAdapter();
    //select
    SqlCommand command = new SqlCommand("SELECT id,fam,name FROM Person", (SqlConnection) _connection);
    _adapter.SelectCommand = command;
    //insert
    command = new SqlCommand(@"INSERT INTO Person (fam, name)
                            VALUES (@fam, @name); SET @id = @@IDENTITY;",(SqlConnection) _connection);
    command.Parameters.Add("@fam", SqlDbType.NVarChar, -1, "fam");
    command.Parameters.Add("@name", SqlDbType.NVarChar, -1, "name");
    SqlParameter parameter = command.Parameters.Add("@id", SqlDbType.Int, 0, "id");
    parameter.Direction = ParameterDirection.Output;
    _adapter.InsertCommand = command;
    //update
    command = new SqlCommand(@"UPDATE Person SET fam = @fam,
                                                name = @name
                                                WHERE (id = @id)", (SqlConnection) _connection);
    command.Parameters.Add("@fam", SqlDbType.NVarChar, -1, "fam");
    command.Parameters.Add("@name", SqlDbType.NVarChar, -1, "name");
    parameter = command.Parameters.Add("@id", SqlDbType.Int, 0, "id");
    parameter.SourceVersion = DataRowVersion.Original;
    _adapter.UpdateCommand = command;
    //delete
    command = new SqlCommand(@"DELETE FROM Person WHERE id = @id", (SqlConnection) _connection);
    parameter = command.Parameters.Add("@id", SqlDbType.Int, 0, "id");
    parameter.SourceVersion = DataRowVersion.Original;
    _adapter.DeleteCommand = command;
    //set source
    _dataSet = new DataSet();
    _adapter.Fill(_dataSet);
    DataGridDataBase.DataContext = _dataSet.Tables[0].DefaultView;
}
private void ButtonAdd_OnClick(object sender, RoutedEventArgs e)
{
    DataRow newRow = _dataSet.Tables[0].NewRow();
    EditWindow editWindow = new EditWindow(newRow);
    editWindow.ShowDialog();
    if (editWindow.DialogResult.HasValue && editWindow.DialogResult.Value)
    {
        _dataSet.Tables[0].Rows.Add(newRow);
        _adapter.Update(_dataSet);
    }
}
private void ButtonEdit_OnClick(object sender, RoutedEventArgs e)
{
    DataRowView editRow = (DataRowView)DataGridDataBase.SelectedItem;
    if (editRow == null)
        return;
    editRow.BeginEdit();
    EditWindow editWindow = new EditWindow(editRow.Row);
    editWindow.ShowDialog();
    if (editWindow.DialogResult.HasValue && editWindow.DialogResult.Value)
    {
        editRow.EndEdit();
        _adapter.Update(_dataSet);
    }
    else
    {
        editRow.CancelEdit();
    }
}
private void ButtonDelete_OnClick(object sender, RoutedEventArgs e)
{
    DataRowView delRow = (DataRowView)DataGridDataBase.SelectedItem;
    if (delRow == null)
        return;
    delRow.Row.Delete();
    _adapter.Update(_dataSet);
}
//дочернее окно:
<Grid.RowDefinitions>
    <RowDefinition Height="4*"/>
    <RowDefinition/>
</Grid.RowDefinitions>
<StackPanel>
    <TextBox x:Name="TextBoxFam" Margin="5"/>
    <TextBox x:Name="TextBoxName" Margin="5"/>
</StackPanel>
<Button Grid.Row="1" x:Name="ButtonOk" Margin="23,19,186,21" Content="OK"/>
<Button Grid.Row="1" x:Name="ButtonCancel" Margin="181,19,28,21" Content="Отмена"/>
private DataRow _dataRow;
public EditWindow(DataRow dataRow)
{
    InitializeComponent();
    _dataRow = dataRow;
}
private void EditWindow_OnLoaded(object sender, RoutedEventArgs e)
{
    TextBoxFam.Text = _dataRow["fam"].ToString();
    TextBoxName.Text = _dataRow["name"].ToString();
    ButtonOk.Click += delegate
    {
        _dataRow["fam"] = TextBoxFam.Text;
        _dataRow["name"] = TextBoxName.Text;
        DialogResult = true;
    };
    ButtonCancel.Click += delegate { DialogResult = false; };
}
```
Код на замену выше показанному - вариант с автоформированием команд:
```csharp
//главное окно
...
private void MainWindow_OnLoaded(object sender, RoutedEventArgs e)
{
    var connectionStringBuilder = new SqlConnectionStringBuilder
    {
        DataSource = @"(localdb)\MSSQLLocalDB",
        InitialCatalog = "sample",
    };
    _connection = new SqlConnection(connectionStringBuilder.ConnectionString);
    _adapter = new SqlDataAdapter();
    //select
    SqlCommand command = new SqlCommand("SELECT id,fam,name FROM Person", (SqlConnection) _connection);
    _adapter.SelectCommand = command;
    //set source
    _dataSet = new DataSet();
    _adapter.Fill(_dataSet);
    DataGridDataBase.DataContext = _dataSet.Tables[0].DefaultView;
}
private void UpdateTable()
{
    _table.Clear();
    _adapter.Fill(_table);
}
private void ButtonAdd_OnClick(object sender, RoutedEventArgs e)
{
    DataRow newRow = _table.NewRow();
    EditWindow editWindow = new EditWindow(newRow);
    editWindow.ShowDialog();
    if (editWindow.DialogResult.HasValue && editWindow.DialogResult.Value)
    {
        _table.Rows.Add(newRow);
        var _ = new SqlCommandBuilder((SqlDataAdapter) _adapter);
        _adapter.Update(_table);
    }
    UpdateTable();
}
private void ButtonEdit_OnClick(object sender, RoutedEventArgs e)
{
    DataRowView editRow = (DataRowView)DataGridDataBase.SelectedItem;
    if (editRow == null)
        return;
    editRow.BeginEdit();
    EditWindow editWindow = new EditWindow(editRow.Row);
    editWindow.ShowDialog();
    if (editWindow.DialogResult.HasValue && editWindow.DialogResult.Value)
    {
        editRow.EndEdit();
        var _ = new SqlCommandBuilder((SqlDataAdapter) _adapter);
        _adapter.Update(_table);
    }
    else
    {
        editRow.CancelEdit();
    }
    UpdateTable();
}
private void ButtonDelete_OnClick(object sender, RoutedEventArgs e)
{
    DataRowView delRow = (DataRowView)DataGridDataBase.SelectedItem;
    if (delRow == null)
        return;
    delRow.Row.Delete();
    var _ = new SqlCommandBuilder((SqlDataAdapter) _adapter);
    _adapter.Update(_table);
    UpdateTable();
}
```

## Простая библиотека, работающая с базой данных

Табличка:
```csharp
CREATE TABLE [dbo].[Inventory](
	[CarId] [int] IDENTITY(1,1) NOT NULL,
	[Make] [nvarchar](50) NULL,
	[Color] [nvarchar](50) NULL,
	[Name] [nvarchar](50) NULL,
    CONSTRAINT [PK_Inventory] PRIMARY KEY ([CarId])
);
```
Процедура:
```csharp
CREATE PROCEDURE [dbo].[GetPetsName] 
@carID int,
@name char(10) output
AS
SELECT @name = Name from Inventory where CarId = @carID;
```

Модель:
```csharp
namespace AutoLotDAL.Modes
{
    public class Car
    {
        public int CarId { get; set; }
        public string Make { get; set; }
        public string Color { get; set; }
        public string Name { get; set; }
    }
```

Класс доступа к данным:
```csharp
namespace AutoLotDAL.DataOperations
{
    public class InventoryDAL
    {
        private readonly string _connectionString;
        public InventoryDAL() : this(@"Data Source=(localdb)\MSSQLLocalDB;Initial Catalog=AutoLot;Integrated Security=True") { }
        public InventoryDAL(string connectionString)
        {
            _connectionString = connectionString;
        }
        private SqlConnection _sqlConnection = null;
        private void OpenConnection()
        {
            _sqlConnection = new SqlConnection { ConnectionString = _connectionString };
            _sqlConnection.Open();
        }
        private void CloseConnection()
        {
            if (_sqlConnection?.State != ConnectionState.Closed)
                _sqlConnection.Close();
        }
        /// <summary> Все автомобили </summary>
        public List<Car> GetAllInventory()
        {
            OpenConnection();
            List<Car> inventory = new List<Car>();
            string str = "SELECT * FROM Inventory";
            using (SqlCommand command = new SqlCommand(str, _sqlConnection))
            {
                SqlDataReader reader = command.ExecuteReader();
                while (reader.Read())
                {
                    inventory.Add(new Car
                    {
                        CarId = (int)reader["CarId"],
                        Make = (string)reader["Make"],
                        Color = (string)reader["Color"],
                        Name = (string)reader["Name"],
                    });
                }
                reader.Close();
            }
            return inventory;
        }
        /// <summary> Один автомобиль </summary>
        public Car GetCar(int id)
        {
            OpenConnection();
            Car car = null;
            string sql = $"SELECT * FROM Inventory WHERE CarId = @id";
            using (SqlCommand command = new SqlCommand(sql, _sqlConnection))
            {
                command.Parameters.Add("@id", SqlDbType.Int, -1).Value = id;
                SqlDataReader reader = command.ExecuteReader(CommandBehavior.CloseConnection);
                while (reader.Read())
                {
                    car = new Car
                    {
                        CarId = (int)reader["CarId"],
                        Make = (string)reader["Make"],
                        Color = (string)reader["Color"],
                        Name = (string)reader["Name"],
                    };
                }
                reader.Close();
            }
            return car;
        }
        /// <summary> Добавление автомобиля </summary>
        public void InsertCar(string make, string color, string name)
        {
            OpenConnection();
            string sql = $"INSERT INTO Inventory (Make, Color, Name) VALUES (@Make, @Color, @Name)";
            using (SqlCommand command = new SqlCommand(sql, _sqlConnection))
            {
                command.Parameters.Add("@Make", SqlDbType.NVarChar, 50).Value = make;
                command.Parameters.Add("@Color", SqlDbType.NVarChar, 50).Value = color;
                command.Parameters.Add("@Name", SqlDbType.NVarChar, 50).Value = name;
                command.ExecuteNonQuery();
            }
            CloseConnection();
        }
        /// <summary> Добавление автомобиля </summary>
        public void InsertCar(Car car)
        {
            OpenConnection();
            string sql = $"INSERT INTO Inventory (Make, Color, Name) VALUES (@Make, @Color, @Name)";
            using (SqlCommand command = new SqlCommand(sql, _sqlConnection))
            {
                command.Parameters.Add("@Make", SqlDbType.NVarChar, 50).Value = car.Make;
                command.Parameters.Add("@Color", SqlDbType.NVarChar, 50).Value = car.Color;
                command.Parameters.Add("@Name", SqlDbType.NVarChar, 50).Value = car.Name;
                command.ExecuteNonQuery();
            }
            CloseConnection();
        }
        /// <summary> Обновление имени автомобиля </summary>
        /// <param name="id"></param>
        public void UpdateCarName(int id, string newName)
        {
            OpenConnection();
            string sql = $"UPDATE Inventory SET Name = @Name WHERE CarId = @id";
            using (SqlCommand command = new SqlCommand(sql, _sqlConnection))
            {
                command.Parameters.Add("@Name", SqlDbType.NVarChar, 50).Value = newName;
                command.Parameters.Add("@id", SqlDbType.Int, -1).Value = id;
                command.ExecuteNonQuery();
            }
            CloseConnection();
        }
        /// <summary> Удаление автомобиля </summary>
        public int DeleteCar(int id)
        {
            int retMe;
            OpenConnection();
            string sql = $"DELETE FROM Inventory WHERE CarId = @id";
            using (SqlCommand command = new SqlCommand(sql, _sqlConnection))
            {
                command.Parameters.Add("@id", SqlDbType.Int, -1).Value = id;
                try
                {
                    retMe = command.ExecuteNonQuery();
                }
                catch (Exception ex)
                {
                    throw new ApplicationException("Этот автомобиль уже заказан!", ex);
                }
            }
            CloseConnection();
            return retMe;
        }
        /// <summary> Получение имени автомобиля </summary>
        public string GetCarName(int carId)
        {
            OpenConnection();
            string carName = default;
            using (SqlCommand command = new SqlCommand("GetPetsName", _sqlConnection))
            {
                command.CommandType = CommandType.StoredProcedure;
                SqlParameter parameter = new SqlParameter
                {
                    ParameterName = "@carID",
                    SqlDbType = SqlDbType.Int,
                    Value = carId,
                    Direction = ParameterDirection.Input,
                };
                command.Parameters.Add(parameter);
                parameter = new SqlParameter
                {
                    ParameterName = "@name",
                    SqlDbType = SqlDbType.Char,
                    Size = 10,
                    Direction = ParameterDirection.Output,
                };
                command.Parameters.Add(parameter);
                command.ExecuteNonQuery();
                carName = (string)command.Parameters["@name"].Value;
                CloseConnection();
            }
            return carName;
        }
    }
```
Дикое использование класса:

```csharp
static void Main(string[] args)
{
    InventoryDAL inventory = new InventoryDAL();
    Console.WriteLine("Все:");
    var list = inventory.GetAllInventory();
    Console.WriteLine("Ид\tМарка\tЦвет\tНазвание");
    foreach(var el in list)
        Console.WriteLine($"{el.CarId}\t{el.Make}\t{el.Color}\t{el.Name}");
    Console.WriteLine();
    try
    {
        var count = inventory.DeleteCar(5);
        if (count != 0)
            Console.WriteLine("Автомобиль удален");
        else
            Console.WriteLine("Автомобиль не удален, так как не найден!");
    }
    catch (Exception ex)
    {
        Console.WriteLine("Автомобиль не удален");
        Console.WriteLine(ex.Message);
    }
    var car = inventory.GetCar(1);
    Console.WriteLine($"Один автомобиль: {car.Make} {car.Name} {car.Color}");
    inventory.InsertCar(new Car {Make = "One", Color = "Gray", Name = "Test"});
    list = inventory.GetAllInventory();
    var newCar = list.First(l => l.Name == "Test").CarId;
    inventory.UpdateCarName(newCar, "Новое имя");
    Console.WriteLine("После добавления и изменения:");
    list = inventory.GetAllInventory();
    Console.WriteLine("Ид\tМарка\tЦвет\tНазвание");
    foreach(var el in list)
        Console.WriteLine($"{el.CarId}\t{el.Make}\t{el.Color}\t{el.Name}");
    Console.WriteLine();
    var Name = inventory.GetCarName(newCar);
    Console.WriteLine($"Имя автомобиля: {Name}");
    inventory.DeleteCar(newCar);
    Console.WriteLine("После удаления:");
    list = inventory.GetAllInventory(); 
    Console.WriteLine("Ид\tМарка\tЦвет\tНазвание");
    foreach(var el in list)
        Console.WriteLine($"{el.CarId}\t{el.Make}\t{el.Color}\t{el.Name}");
    Console.ReadKey();
```



