# Основы WPF Базы данных

## Получение данных из базы данных

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

Пример использования адаптера с авто написанным запросом SqlCommandBuilder:
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
        DataSet ds = new DataSet();
        adapter.Fill(ds);                
        DataTable dt = ds.Tables[0];
        DataRow newRow = dt.NewRow();
        newRow["FIO"] = "Тестов Тест Тестович";
        newRow["Birthday"] = "25.05.2002";
        dt.Rows.Add(newRow);
        SqlCommandBuilder commandBuilder = new SqlCommandBuilder(adapter);
        adapter.Update(ds);
    }
}
```

## Приложение работы с базой данных

Пример простого приложения работы с базой данных на ADO:

Окно MainWindow.xaml:
```csharp
<Window x:Class="WpfApp1HelloWPF.MainWindow"
...
        Title="Заголовок окна" Height="500" Width="600" WindowStartupLocation="CenterScreen" MinHeight="400" MinWidth="400" Loaded="Window_Loaded">
    <Grid HorizontalAlignment="Stretch">
        <Grid.RowDefinitions>
            <RowDefinition Height="5*"/>
            <RowDefinition/>
        </Grid.RowDefinitions>
        <Grid.ColumnDefinitions>
            <ColumnDefinition/>
            <ColumnDefinition/>
            <ColumnDefinition/>
        </Grid.ColumnDefinitions>
        <DataGrid x:Name="peopleDataGrid" Grid.ColumnSpan="3" AutoGenerateColumns="False" EnableRowVirtualization="True" ItemsSource="{Binding}" Margin="10" HorizontalAlignment="Stretch" IsReadOnly="True" SelectionMode="Single">
            <DataGrid.Columns>
                <DataGridTextColumn x:Name="idColumn" Binding="{Binding Id}" Header="Id" IsReadOnly="True" Width="Auto"/>
                <DataGridTextColumn x:Name="fioColumn" Binding="{Binding FIO}" Header="ФИО" IsReadOnly="True" Width="Auto"/>
                <DataGridTextColumn x:Name="birthdayColumn" Binding="{Binding Birthday}" Header="День рождения" IsReadOnly="True" Width="Auto"/>
                <DataGridTextColumn x:Name="emailColumn" Binding="{Binding Email}" Header="Email" IsReadOnly="True" Width="Auto"/>
                <DataGridTextColumn x:Name="phoneColumn" Binding="{Binding Phone}" Header="Телефон" IsReadOnly="True" Width="Auto"/>
            </DataGrid.Columns>
        </DataGrid>
        <Button x:Name="addButton" Content="Добавить" Grid.Column="0" Grid.Row="1" MaxWidth="300" MinWidth="50" Height="40" Margin="10" Click="addButton_Click"/>
        <Button x:Name="updateButton" Content="Изменить" Grid.Column="1" Grid.Row="1" MaxWidth="300" MinWidth="50" Height="40" Margin="10" Click="updateButton_Click"/>
        <Button x:Name="deleteButton" Content="Удалить" Grid.Column="2" Grid.Row="1" MaxWidth="300" MinWidth="50" Height="40" Margin="10" Click="deleteButton_Click"/>
    </Grid>
</Window>
public partial class MainWindow : Window
{
    SqlConnection connection;
    SqlDataAdapter adapter;
    DataTable dt;
    public MainWindow()
    {
        InitializeComponent();
    }
    private void Window_Loaded(object sender, RoutedEventArgs e)
    {
        string connectionString = @"data source = DESKTOP-Q5PLE8H\SQLEXPRESS; Initial Catalog = Lesson7; Integrated Security = True";
        connection = new SqlConnection(connectionString);
        adapter = new SqlDataAdapter();
        SqlCommand selectCommand = new SqlCommand(@"SELECT Id,FIO,Birthday,Email,Phone FROM People",connection);
        adapter.SelectCommand = selectCommand;
        SqlCommand insertCommand = new SqlCommand(@"INSERT INTO People (FIO,Birthday,Email,Phone) VALUES (@FIO,@Birthday,@Email,@Phone); SET @ID = @@IDENTITY;",connection);
        insertCommand.Parameters.Add("@FIO", SqlDbType.VarChar,256,"FIO");
        insertCommand.Parameters.Add("@Birthday", SqlDbType.VarChar,256,"Birthday");
        insertCommand.Parameters.Add("@Email", SqlDbType.VarChar,100,"Email");
        insertCommand.Parameters.Add("@Phone", SqlDbType.VarChar,100,"Phone");
        var insertParam = insertCommand.Parameters.Add("@ID",SqlDbType.Int,0,"ID");
        insertParam.Direction = ParameterDirection.Output;
        adapter.InsertCommand = insertCommand;
        SqlCommand updateCommand = new SqlCommand(@"UPDATE People SET FIO=@FIO,Birthday=@Birthday,Email=@Email,Phone=@Phone WHERE ID=@ID",connection);
        updateCommand.Parameters.Add("@FIO", SqlDbType.VarChar,256,"FIO");
        updateCommand.Parameters.Add("@Birthday", SqlDbType.VarChar,256,"Birthday");
        updateCommand.Parameters.Add("@Email", SqlDbType.VarChar,100,"Email");
        updateCommand.Parameters.Add("@Phone", SqlDbType.VarChar,100,"Phone");
        var updateParam = updateCommand.Parameters.Add("@ID",SqlDbType.Int,0,"ID");
        updateParam.SourceVersion = DataRowVersion.Original;
        adapter.UpdateCommand = updateCommand;
        SqlCommand deleteCommand = new SqlCommand(@"DELETE FROM People WHERE ID=@ID",connection);
        var deleteParam = deleteCommand.Parameters.Add("@ID",SqlDbType.Int,0,"ID");
        deleteParam.SourceVersion = DataRowVersion.Original;
        adapter.DeleteCommand = deleteCommand;
        dt = new DataTable();
        adapter.Fill(dt);
        peopleDataGrid.DataContext = dt.DefaultView;
    }
    private void addButton_Click(object sender, RoutedEventArgs e)
    {
        DataRow newRow = dt.NewRow();
        EditWindow editWindow = new EditWindow(newRow);
        editWindow.ShowDialog();
        if (editWindow.DialogResult.HasValue && editWindow.DialogResult.Value)
        {
            dt.Rows.Add(editWindow.resultRow);
            adapter.Update(dt);
        }
    }
    private void updateButton_Click(object sender, RoutedEventArgs e)
    {
        DataRowView newRow = (DataRowView)peopleDataGrid.SelectedItem;
        if (newRow == null)
            return;
        newRow.BeginEdit();
        EditWindow editWindow = new EditWindow(newRow.Row);
        editWindow.ShowDialog();
        if (editWindow.DialogResult.HasValue && editWindow.DialogResult.Value)
        {
            newRow.EndEdit();
            adapter.Update(dt);
        }
        else
        {
            newRow.CancelEdit();
        }
    }
    private void deleteButton_Click(object sender, RoutedEventArgs e)
    {
        DataRowView newRow = (DataRowView)peopleDataGrid.SelectedItem;
        if (newRow == null)
            return;
        newRow.Row.Delete();
        adapter.Update(dt);
    }
}
```
Окно EditWindow.xaml:
```csharp
<Window x:Class="WpfApp1HelloWPF.EditWindow"
...
        Title="Окно редактирования" Height="300" Width="400" WindowStartupLocation="CenterScreen" Loaded="Window_Loaded">
    <Grid>
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="*"/>
            <ColumnDefinition Width="*"/>
        </Grid.ColumnDefinitions>
        <Grid.RowDefinitions>
            <RowDefinition Height="4*"/>
            <RowDefinition Height="AUto"/>
        </Grid.RowDefinitions>
        <Grid x:Name="editGrid" Grid.ColumnSpan="2">
            <Grid.ColumnDefinitions>
                <ColumnDefinition Width="Auto"/>
                <ColumnDefinition Width="Auto"/>
            </Grid.ColumnDefinitions>
            <Grid.RowDefinitions>
                <RowDefinition Height="Auto"/>
                <RowDefinition Height="Auto"/>
                <RowDefinition Height="Auto"/>
                <RowDefinition Height="Auto"/>
                <RowDefinition Height="Auto"/>
            </Grid.RowDefinitions>
            <Label Content="ФИО:" Grid.Column="0" HorizontalAlignment="Left" Margin="3" Grid.Row="1" VerticalAlignment="Center"/>
            <TextBox x:Name="fioTextBox" Grid.Column="1" HorizontalAlignment="Left" Height="23" Margin="3" Grid.Row="1" VerticalAlignment="Center" Width="270"/>
            <Label Content="День рождения:" Grid.Column="0" HorizontalAlignment="Left" Margin="3" Grid.Row="2" VerticalAlignment="Center"/>
            <TextBox x:Name="birthdayTextBox" Grid.Column="1" HorizontalAlignment="Left" Height="23" Margin="3" Grid.Row="2" VerticalAlignment="Center" Width="270"/>
            <Label Content="Почта:" Grid.Column="0" HorizontalAlignment="Left" Margin="3" Grid.Row="3" VerticalAlignment="Center"/>
            <TextBox x:Name="emailTextBox" Grid.Column="1" HorizontalAlignment="Left" Height="23" Margin="3" Grid.Row="3" VerticalAlignment="Center" Width="270"/>
            <Label Content="Телефон:" Grid.Column="0" HorizontalAlignment="Left" Margin="3" Grid.Row="4" VerticalAlignment="Center"/>
            <TextBox x:Name="phoneTextBox" Grid.Column="1" HorizontalAlignment="Left" Height="23" Margin="3" Grid.Row="4" VerticalAlignment="Center" Width="270"/>
        </Grid>
        <Button x:Name="saveButton" IsDefault="True" Content="Сохранить" Margin="10" Grid.Row="1" HorizontalAlignment="Center" VerticalAlignment="Bottom" Width="120" Height="30" Click="saveButton_Click"/>
        <Button x:Name="cancelButton" IsCancel="True" Content="Отмена" Margin="10" Grid.Row="1" Grid.Column="1" HorizontalAlignment="Center" VerticalAlignment="Bottom" Width="120" Height="30" Click="cancelButton_Click"/>
    </Grid>
</Window>
public partial class EditWindow : Window
{
    public DataRow resultRow {get;set;}
    public EditWindow(DataRow dataRow)
    {
        InitializeComponent();
        resultRow = dataRow;
    }
    private void Window_Loaded(object sender, RoutedEventArgs e)
    {
        fioTextBox.Text = resultRow["FIO"].ToString();
        birthdayTextBox.Text = resultRow["Birthday"].ToString();
        emailTextBox.Text = resultRow["Email"].ToString();
        phoneTextBox.Text = resultRow["Phone"].ToString();
    }
    private void saveButton_Click(object sender, RoutedEventArgs e)
    {
        resultRow["FIO"] = fioTextBox.Text;
        resultRow["Birthday"] = birthdayTextBox.Text;
        resultRow["Email"] = emailTextBox.Text;
        resultRow["Phone"] = phoneTextBox.Text;
        DialogResult = true;
    }
    private void cancelButton_Click(object sender, RoutedEventArgs e)
    {
        DialogResult = false;
    }
}
```




