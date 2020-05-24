# Основы WPF Приложение Базы данных

Пример обычного приложения работы с базой данных:

Таблицы базы данных:
```sql
CREATE TABLE Departament
(
Id INT IDENTITY(1,1) NOT NULL,
Departament VARCHAR(128) NOT NULL,
CONSTRAINT PK_Id_Departament PRIMARY KEY (Id),
);
INSERT INTO Departament(Departament)
VALUES ('Отдел управления'),('Отдел финансов'),('Отдел рекламы');
CREATE TABLE Employee
(
Id INT IDENTITY(1,1) NOT NULL,
Fam VARCHAR(256) NOT NULL,
Name VARCHAR(256) NOT NULL,
Age INT NOT NULL,
Salary INT NOT NULL,
DepartamentId INT NOT NULL,
CONSTRAINT PK_Id_Employee PRIMARY KEY (Id),
CONSTRAINT FK_OrPrOrderID FOREIGN KEY (DepartamentId)
	REFERENCES Departament(Id),
);
INSERT INTO [Employee](Fam,Name,Age,Salary,DepartamentId)
VALUES ('Иванов','Иван',18,10000,1),('Федотов','Федот',19,11000,1),('Петров','Петр',18,9000,1),('Яролов','Ян',20,15000,2),('Агафонова','Надя',19,12000,2),('Бибикова','Надежда',29,13000,3);
```

Файл "App.config":
```csharp
...
  <connectionStrings>
    <add name="DefaultConnection" connectionString="data source = DESKTOP-Q5PLE8H\SQLEXPRESS; Initial Catalog = Lesson7; Integrated Security = True; Connect Timeout = 3" providerName="System.Data.SqlClient"/> //ресурс с подключением
  </connectionStrings>
...
```

Окно "MainWindow.xaml":
```csharp
<Window x:Class="WpfApp1Company.MainWindow"
...
        Loaded="Window_Loaded" Title="Geekbrains C# Уровень 2. Методичка 7. Задачи № 1,2,3,4." Height="500" Width="600" MinHeight="300" MinWidth="300" Background="#FFCFFBBA" WindowStartupLocation="CenterScreen" Icon="Avatar.ico">
    <Grid>
        <Grid.ColumnDefinitions><ColumnDefinition/><ColumnDefinition/><ColumnDefinition/></Grid.ColumnDefinitions>
        <Grid.RowDefinitions><RowDefinition Height="40"/><RowDefinition Height="5*"/><RowDefinition/></Grid.RowDefinitions>
        <Label Content="WPF-приложение для компании с базой данных на борту." Height="40" HorizontalContentAlignment="Center" VerticalAlignment="Top" FontSize="20" Background="LightGreen" Grid.ColumnSpan="3"/>
        //отображение двух таблиц с внутренней связью из базы данных
        <DataGrid ItemsSource="{Binding}" x:Name="viewEmployeeDataGrid" Grid.ColumnSpan="3" AutoGenerateColumns="False" EnableRowVirtualization="True" Margin="5" IsReadOnly="True" Grid.Row="1" SelectionMode="Single" FontSize="14">
            <DataGrid.Columns>
                <DataGridTextColumn Binding="{Binding Fam}" x:Name="famColumn" Header="Фамилия" IsReadOnly="True" Width="Auto"/>
                <DataGridTextColumn Binding="{Binding Name}" x:Name="nameColumn" Header="Имя" IsReadOnly="True" Width="Auto"/>
                <DataGridTextColumn Binding="{Binding Age}" x:Name="ageColumn" Header="Возраст" IsReadOnly="True" Width="Auto"/>
                <DataGridTextColumn Binding="{Binding Salary}" x:Name="salaryColumn" Header="Зарплата" IsReadOnly="True" Width="Auto"/>
                <DataGridTextColumn Binding="{Binding Departament}" x:Name="departamentColumn" Header="Отдел" IsReadOnly="True" Width="Auto"/>
            </DataGrid.Columns>
        </DataGrid>
        //кнопка открытия окна редактирования
        <Button Click="employeesButton_Click" x:Name="employeesButton" Content="Сотрудники" MaxHeight="50" MaxWidth="200" Grid.Column="0" Grid.Row="2" Margin="5" FontSize="14"/>
    </Grid>
</Window>
public partial class MainWindow : Window
{
    public static SqlConnection Connection; //одно содение на все приложение
    SqlDataAdapter adapter;
    DataTable table;
    public MainWindow()
    {
        InitializeComponent();
    }
    private void Window_Loaded(object sender, RoutedEventArgs e)
    {
        try
        {
            string connectionString = ConfigurationManager.ConnectionStrings["DefaultConnection"].ConnectionString;
            Connection = new SqlConnection(connectionString);
            adapter = new SqlDataAdapter();
            SqlCommand selectCommand = new SqlCommand("SELECT e.Id,e.Fam,e.Name,e.Age,e.Salary,d.Departament FROM Employee e INNER JOIN Departament d ON e.DepartamentId=d.Id;",Connection);
            adapter.SelectCommand = selectCommand;
            table = new DataTable();
            adapter.Fill(table);
        }
        catch (Exception ex)
        {
            MessageBox.Show("Ошибка работы с базой данных:\n"+ex.Message);
        };
        viewEmployeeDataGrid.DataContext = table.DefaultView;
    }
    private void employeesButton_Click(object sender, RoutedEventArgs e)
    {
        EmployeeWindow employeeWindow = new EmployeeWindow();
        employeeWindow.ShowDialog();
        table.Clear();
        adapter.Fill(table);
    }
}
```
Окно редактирования сотрудников "EmployeeWindow.xaml":
```csharp
<Window x:Class="WpfApp1Company.EmployeeWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:WpfApp1Company"
        mc:Ignorable="d"
        Loaded="Window_Loaded" Title="Редактирование сотрудников" Height="600" Width="500" MinHeight="300" MinWidth="300" Background="#FFBDFCFF" WindowStartupLocation="CenterScreen" Icon="Avatar.ico">
    <Grid>
        <Grid.ColumnDefinitions><ColumnDefinition/><ColumnDefinition/><ColumnDefinition/></Grid.ColumnDefinitions>
        <Grid.RowDefinitions><RowDefinition Height="6*"/><RowDefinition/></Grid.RowDefinitions>
        <DataGrid x:Name="employeesDataGrid" Grid.ColumnSpan="3" AutoGenerateColumns="False" EnableColumnVirtualization="True" ItemsSource="{Binding}" Margin="5" IsReadOnly="True" SelectionMode="Single" FontSize="14">
            <DataGrid.Columns>
                <DataGridTextColumn x:Name="famColumn" Binding="{Binding Fam}" Header="Фамилия" IsReadOnly="True" Width="Auto"/>
                <DataGridTextColumn x:Name="nameColumn" Binding="{Binding Name}" Header="Имя" IsReadOnly="True" Width="Auto"/>
                <DataGridTextColumn x:Name="ageColumn" Binding="{Binding Age}" Header="Возраст" IsReadOnly="True" Width="Auto"/>
                <DataGridTextColumn x:Name="salaryColumn" Binding="{Binding Salary}" Header="Зарплата" IsReadOnly="True" Width="Auto"/>
                <DataGridTextColumn x:Name="departamentIdColumn" Binding="{Binding DepartamentId}" Header="Ид отдела" IsReadOnly="True" Width="Auto"/>
            </DataGrid.Columns>
        </DataGrid>
        <Button Click="addButton_Click" x:Name="addButton" Content="Добавить" Grid.Column="0" Grid.Row="1" MaxWidth="300" MaxHeight="50" Margin="5" FontSize="14"/>
        <Button Click="editButton_Click" x:Name="editButton" Content="Изменить" Grid.Column="1" Grid.Row="1" MaxWidth="300" MaxHeight="50" Margin="5" FontSize="14"/>
        <Button Click="deleteButton_Click" x:Name="deleteButton" Content="Удалить" Grid.Column="2" Grid.Row="1" MaxWidth="300" MaxHeight="50" Margin="5" FontSize="14"/>
    </Grid>
</Window>
public partial class EmployeeWindow : Window
{
    SqlDataAdapter adapter;
    DataTable table;
    public EmployeeWindow()
    {
        InitializeComponent();
    }
    private void Window_Loaded(object sender, RoutedEventArgs e)
    {
        adapter = new SqlDataAdapter();
        SqlCommand selectCommand = new SqlCommand("SELECT Id,Fam,Name,Age,Salary,DepartamentId FROM Employee;",MainWindow.Connection);
        adapter.SelectCommand = selectCommand;
        table = new DataTable();
        adapter.Fill(table);
        employeesDataGrid.DataContext = table.DefaultView;
    }
    private void addButton_Click(object sender, RoutedEventArgs e)
    {
        DataRow newRow = table.NewRow();
        EmployeeEditWindow employeeEditWindow = new EmployeeEditWindow(newRow);
        employeeEditWindow.Title = "Добавление нового сотрудника";
        employeeEditWindow.ShowDialog();
        if (employeeEditWindow.DialogResult.HasValue && employeeEditWindow.DialogResult.Value)
        {
            table.Rows.Add(employeeEditWindow.ResultRow);
            SqlCommandBuilder commandBuilder = new SqlCommandBuilder(adapter);
            try
            {
                adapter.Update(table);
            }
            catch (Exception ex)
            {
                MessageBox.Show("Ошибка при добавлении сотрудника:\n"+ex.Message);
            }
        }
    }
    private void editButton_Click(object sender, RoutedEventArgs e)
    {
        DataRowView newRow = (DataRowView)employeesDataGrid.SelectedItem;
        if (newRow == null)
        {
            MessageBox.Show("Выделите сотрудника, которого нужно изменить!");
            return;
        }
        newRow.BeginEdit();
        EmployeeEditWindow employeeEditWindow = new EmployeeEditWindow(newRow.Row);
        employeeEditWindow.Title = "Изменение выбранного сотрудника";
        employeeEditWindow.ShowDialog();
        if (employeeEditWindow.DialogResult.HasValue && employeeEditWindow.DialogResult.Value)
        {
            newRow.EndEdit();
            SqlCommandBuilder commandBuilder = new SqlCommandBuilder(adapter);
            try
            {
                adapter.Update(table);
            }
            catch (Exception ex)
            {
                MessageBox.Show("Ошибка при изменении сотрудника:\n"+ex.Message);
            }
        }
        else
        {
            newRow.CancelEdit();
        }
    }
    private void deleteButton_Click(object sender, RoutedEventArgs e)
    {
        DataRowView newRow = (DataRowView)employeesDataGrid.SelectedItem;
        if (newRow == null)
        {
            MessageBox.Show("Выделите сотрудника, которого нужно удалить!");
            return;
        }
        newRow.Delete();
        SqlCommandBuilder commandBuilder = new SqlCommandBuilder(adapter);
        try
        {
            adapter.Update(table);
        }
        catch (Exception ex)
        {
            MessageBox.Show("Ошибка при удалении сотрудника:\n"+ex.Message);
        }
        table.Clear();
        adapter.Fill(table);
    }
}
```
Окно редактирования отдельного сотрудника "EmployeeEditWindow.xaml":
```csharp
<Window x:Class="WpfApp1Company.EmployeeEditWindow"
...
        Loaded="Window_Loaded" Title="Редактирование сотрудника" Height="400" Width="500" MinHeight="200" MinWidth="300" Background="#FFBDFCFF" WindowStartupLocation="CenterScreen" Icon="Avatar.ico">
    <Grid>
        <Grid.ColumnDefinitions><ColumnDefinition/><ColumnDefinition/></Grid.ColumnDefinitions>
        <Grid.RowDefinitions><RowDefinition Height="5*"/><RowDefinition/></Grid.RowDefinitions>
        <Grid Grid.ColumnSpan="2">
            <Grid.ColumnDefinitions><ColumnDefinition Width="Auto"/><ColumnDefinition Width="Auto"/></Grid.ColumnDefinitions>
            <Grid.RowDefinitions><RowDefinition/><RowDefinition/><RowDefinition/><RowDefinition/><RowDefinition/></Grid.RowDefinitions>
            <Label Content="Фамилия:" Grid.Column="0" HorizontalAlignment="Left" Margin="3" Grid.Row="0" VerticalAlignment="Center" FontSize="14"/>
            <TextBox x:Name="famTextBox" Grid.Column="1" HorizontalAlignment="Left" Height="23" Margin="3" Grid.Row="0" VerticalAlignment="Center" Width="280" FontSize="14"/>
            <Label Content="Имя:" Grid.Column="0" HorizontalAlignment="Left" Margin="3" Grid.Row="1" VerticalAlignment="Center" FontSize="14"/>
            <TextBox x:Name="nameTextBox" Grid.Column="1" HorizontalAlignment="Left" Height="23" Margin="3" Grid.Row="1" VerticalAlignment="Center" Width="280" FontSize="14"/>
            <Label Content="Возраст:" Grid.Column="0" HorizontalAlignment="Left" Margin="3" Grid.Row="2" VerticalAlignment="Center" FontSize="14"/>
            <TextBox x:Name="ageTextBox" Grid.Column="1" HorizontalAlignment="Left" Height="23" Margin="3" Grid.Row="2" VerticalAlignment="Center" Width="280" FontSize="14"/>
            <Label Content="Зарплата:" Grid.Column="0" HorizontalAlignment="Left" Margin="3" Grid.Row="3" VerticalAlignment="Center" FontSize="14"/>
            <TextBox x:Name="salaryTextBox" Grid.Column="1" HorizontalAlignment="Left" Height="23" Margin="3" Grid.Row="3" VerticalAlignment="Center" Width="280" FontSize="14"/>
            <Label Content="Отдел:" Grid.Column="0" HorizontalAlignment="Left" Margin="3" Grid.Row="4" VerticalAlignment="Center" FontSize="14" />
            <ComboBox ItemsSource="{Binding}" x:Name="departamentsComboBox" Grid.Column="1" HorizontalAlignment="Left" Height="23" Margin="3" Grid.Row="4" VerticalAlignment="Center" Width="280" FontSize="14"/>
        </Grid>
        <Button Click="saveButton_Click" x:Name="saveButton" IsDefault="True" Content="Сохранить" Margin="10" Grid.Row="1" FontSize="14"/>
        <Button Click="cancelButton_Click" x:Name="cancelButton" IsCancel="True" Content="Отмена" Margin="10" Grid.Row="1" Grid.Column="1" FontSize="14"/>
    </Grid>
</Window>
public partial class EmployeeEditWindow : Window
{
    public DataRow ResultRow {get;set;}
    SqlDataAdapter adapter;
    DataTable table;
    public EmployeeEditWindow(DataRow dataRow)
    {
        InitializeComponent();
        ResultRow = dataRow;
    }
    private void Window_Loaded(object sender, RoutedEventArgs e)
    {
        famTextBox.Text = ResultRow["Fam"].ToString();
        nameTextBox.Text = ResultRow["Name"].ToString();
        ageTextBox.Text = ResultRow["Age"].ToString();
        salaryTextBox.Text = ResultRow["Salary"].ToString();
        adapter = new SqlDataAdapter();
        SqlCommand selectCommand = new SqlCommand("SELECT Id,Departament FROM Departament;",MainWindow.Connection);
        adapter.SelectCommand = selectCommand;
        table = new DataTable();
        adapter.Fill(table);
        departamentsComboBox.ItemsSource = table.DefaultView;
        departamentsComboBox.DisplayMemberPath = "Departament";
        departamentsComboBox.SelectedValuePath = "Id"; //привязка выбираемых значений
        departamentsComboBox.SelectedValue = ResultRow["DepartamentId"];
    }
    private void saveButton_Click(object sender, RoutedEventArgs e)
    {
        ResultRow["Fam"] = famTextBox.Text;
        ResultRow["Name"] = nameTextBox.Text;
        ResultRow["Age"] = ageTextBox.Text;
        ResultRow["Salary"] = salaryTextBox.Text;
        ResultRow["DepartamentId"] = departamentsComboBox.SelectedValue; //выбранное занение
        DialogResult = true;
    }
    private void cancelButton_Click(object sender, RoutedEventArgs e)
    {
        DialogResult = false;
    }
}
```
