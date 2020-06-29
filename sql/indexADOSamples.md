# ADO.NET Примеры

## Чтение данных

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
















