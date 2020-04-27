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






