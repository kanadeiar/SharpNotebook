# Основы WPF Привязки

## База привязок

Пример использования привязок:
```csharp
<Window x:Class="WpfApp1HelloWPF.MainWindow"
...
        Title="Заголовок окна" Height="400" Width="600" WindowStartupLocation="CenterScreen">
    <StackPanel>
        <TextBox x:Name="textBoxValue" Text=""/>
        <WrapPanel Margin="1">
            <TextBlock Text="Текст: " FontWeight="Bold" />
            <TextBlock Text="{Binding ElementName=textBoxValue, Path=Text}"/> //Привязки
        </WrapPanel>
    </StackPanel>
</Window>
```
Создание привязки в отдельном файле кода:
```csharp
<Window x:Class="WpfApp1HelloWPF.MainWindow"
...
        Title="Заголовок окна" Height="400" Width="600" WindowStartupLocation="CenterScreen">
    <StackPanel>
        <TextBox x:Name="textBoxValue" Text=""/>
        <WrapPanel Margin="1">
            <TextBlock Text="Текст: " FontWeight="Bold" />
            <TextBlock x:Name="textBlockInValue"/>
        </WrapPanel>
    </StackPanel>
</Window>
    public MainWindow()
    {
        InitializeComponent();
        Binding binding = new Binding();
        binding.ElementName="textBoxValue";
        binding.Path=new PropertyPath("Text");
        textBlockInValue.SetBinding(TextBlock.TextProperty, binding);
    }
```

## Форматирование значений привязки

Пример форматирования значений привязки с использованием статического ресурса:
```csharp
<Window x:Class="WpfApp1HelloWPF.MainWindow"
...
        Title="Заголовок окна" Height="400" Width="600" WindowStartupLocation="CenterScreen">
    <Window.Resources>
        <local:Employee x:Key="Employee" Name="Вася" Age="18" Salary="9000"/>
    </Window.Resources>
    <Grid>
        <TextBlock Text="{Binding StringFormat='Зарплата составляет {0} рублей', Source={StaticResource Employee}, Path=Salary}"/>
    </Grid>
</Window>
    class Employee
    {
        public string Name {get;set;}
        public int Age {get;set;}
        public int Salary {get;set;}
    }
```
Заметка: для лечения глюка - перевести тип проекта с х86 на х64 и пересобрать.

Пример конвертирования значения:

```csharp
<Window x:Class="WpfApp1HelloWPF.MainWindow"
...
    Title="Заголовок окна" Height="400" Width="600" WindowStartupLocation="CenterScreen">
    <Window.Resources>
        <local:YesNoToBooleanConverter x:Key="YesNoToBooleanConverter"/>
    </Window.Resources>
    <StackPanel Margin="10">
        <TextBox x:Name="textBoxValue"/>
        <WrapPanel>
            <TextBlock Text="Текущее значение:" />
            <TextBlock Text="{Binding ElementName=textBoxValue, Path=Text, Converter={StaticResource YesNoToBooleanConverter}}"></TextBlock>
        </WrapPanel>
        <CheckBox IsChecked="{Binding ElementName=textBoxValue, Path=Text, Converter={StaticResource YesNoToBooleanConverter}}" Content="Да"/>
    </StackPanel>
</Window>
public class YesNoToBooleanConverter : IValueConverter
{
    public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
    {
        switch(value.ToString().ToLower())
        {
            case "yes":
            case "true":
                return true;
            case "no":
            case "false":
                return false;
        }
        return false;
    }
    public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
    {
        if (value is bool)
        {
            if ((bool)value==true)
                return "yes";
            else
                return "no";
        }
        return "no";
    }
}
```
Привязка к свойствам окна:
```csharp
<Window x:Class="WpfApp1HelloWPF.MainWindow"
...
        Title="Заголовок окна" Height="400" Width="600" WindowStartupLocation="CenterScreen">
    <Window.Resources></Window.Resources>
    <StackPanel Margin="10">
        <WrapPanel>
            <TextBlock Text="Заголовок: "/>
            <TextBlock Text="{Binding Title}"/>
        </WrapPanel>
        <WrapPanel>
            <TextBlock Text="Размеры: "/>
            <TextBlock Text="{Binding Width}"/>
            <TextBlock Text=" х "/>
            <TextBlock Text="{Binding Height}"/>
        </WrapPanel>
    </StackPanel>
</Window>
    public MainWindow()
    {
        InitializeComponent();
        this.DataContext = this;
    }
```

## Связывание данных через наблюдаемые коллекции

Пример связывания данных:
```csharp
<Window x:Class="WpfApp1HelloWPF.MainWindow"
...
        Title="Заголовок окна" Height="400" Width="600" WindowStartupLocation="CenterScreen">
    <DockPanel>
        <StackPanel DockPanel.Dock="Right" Margin="10,0,0,0">
            <Button Name="buttonAddUser" Click="buttonAddUser_Click">Добавить</Button>
            <Button Name="buttonChangeUser" Click="buttonChangeUser_Click" Margin="0,5">Изменить</Button>
            <Button Name="buttonDeleteUser" Click="buttonDeleteUser_Click">Удалить</Button>
        </StackPanel>
        <ListBox Name="listBoxUsers" DisplayMemberPath="Name"></ListBox>
    </DockPanel>
</Window>
public partial class MainWindow : Window
{
    private ObservableCollection<User> users = new ObservableCollection<User>();
    public MainWindow()
    {
        InitializeComponent();
        users.Add(new User{Name = "Петя"});
        users.Add(new User{Name = "Коля"});
        listBoxUsers.ItemsSource = users;
    }
    private void buttonAddUser_Click(object sender, RoutedEventArgs e)
    {
        users.Add(new User{Name = "Вася"});
    }
    private void buttonChangeUser_Click(object sender, RoutedEventArgs e)
    {
        if (listBoxUsers.SelectedItem != null)
            (listBoxUsers.SelectedItem as User).Name = "Иван";
    }
    private void buttonDeleteUser_Click(object sender, RoutedEventArgs e)
    {
        if (listBoxUsers.SelectedItem != null)
            users.Remove(listBoxUsers.SelectedItem as User);
    }
}
public class User : INotifyPropertyChanged
{
    private string name;
    public string Name
    {
        get => name;
        set
        {
            if (name != value)
            {
                name = value;
                NotifyPropertyChanged("Name");
            }
        }
    }
    public event PropertyChangedEventHandler PropertyChanged;
    public void NotifyPropertyChanged(string propName)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propName));
    }
}
```

Пример отладки ошибок связывания данных:

```csharp
<Window x:Class="WpfApp1HelloWPF.MainWindow"
...
        Title="Заголовок окна" Height="400" Width="600" WindowStartupLocation="CenterScreen">
    <Window.Resources>
        <local:DebugDummyConverter x:Key="DebugDummyConverter"/>
    </Window.Resources>
    <Grid Margin="10">
        <TextBlock Text="{Binding Title, ElementName=wnd, Converter={StaticResource DebugDummyConverter}}"/>
    </Grid>
</Window>
public class DebugDummyConverter : IValueConverter
{
    public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
    {
        Debugger.Break();
        return value;
    }
    public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
    {
        Debugger.Break();
        return value;
    }
}
```




