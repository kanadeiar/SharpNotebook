# WPF Привязки (свойства зависимости)

Привязка данных представляет собой действие по подключению свойств элемента управления к значениям данных, изменяющихся на протяжении жизненного цикла приложения. Источником операции привязки данных являются сами данные, а местом назначения - свойство элемента управления пользовательского интерфейса. Инфраструктура WPF дает развитую экосистему привязки данных, способную поддерживаться в разметке. И обеспечивает синхронизацию источника и цели в случае изменения знаечний данных.

Механизмом, обеспечивающим определение привязки в разметке XAML, является расширение разметки {Binding}. Привязки можно определять как посредством кода C#, так и прямо в разметке. Значение ElementName указывает источник операции привязки данных, а Path - свойство к которому осущетсвляется привязка.

Пример установки привязки в разметке:
```csharp
<StackPanel>
    <Label Content="Переместить ползунок"/>
    <ScrollBar x:Name="myScrollBar" Orientation="Horizontal" Height="30" Minimum="1" Maximum="100" LargeChange="1" SmallChange="1"/>
    <!--<Label x:Name="labelThumb" Height="30" BorderBrush="Blue" BorderThickness="2" Content="{Binding Value, ElementName=myScrollBar}"/> это другой вариант -->
    <Label x:Name="labelThumb" Height="30" BorderBrush="Blue" BorderThickness="2" Content="{Binding ElementName=myScrollBar, Path=Value}"/>
</StackPanel>
```
Пример установки привязки в коде C#:
```csharp
<StackPanel>
    <Label Content="Переместить ползунок"/>
    <ScrollBar x:Name="myScrollBar" Orientation="Horizontal" Height="30" Minimum="1" Maximum="100" LargeChange="1" SmallChange="1"/>
    <Label x:Name="labelThumb" Height="30" BorderBrush="Blue" BorderThickness="2" Content="0"/>
</StackPanel>
public MainWindow()
{
    InitializeComponent();
    Binding binding = new Binding
    {
        ElementName = "myScrollBar",
        Path = new PropertyPath("Value"),
    };
    labelThumb.SetBinding(Label.ContentProperty, binding);
}
```
В XAML для определения привязки данных можно использовать альтернативный формати, разбивающий значения, указанные расширенной разметкой {Bunding}, путем установки свойства DataContext. Пример:
```csharp
<Label x:Name="labelThumb" Height="30" BorderBrush="Blue" BorderThickness="2" DataContext="{Binding ElementName=myScrollBar}" Content="{Binding Path=Value}"/>
```
Это может быть удобно из-за того, что подэлементы наследуют свои значения в дереве разметки. Пример использования альтернативного формата:
```csharp
<StackPanel DataContext="{Binding ElementName=myScrollBar}">
    <Label Content="Переместить ползунок:"/>
    <ScrollBar x:Name="myScrollBar" Orientation="Horizontal" Height="30" Minimum="1" Maximum="100" LargeChange="1" SmallChange="1"/>
    <Label x:Name="labelThumb" Height="30" BorderBrush="Blue" BorderThickness="2" Content="{Binding Path=Value}"/>
    <Button Content="Нажми меня" Height="300" FontSize="{Binding Path=Value}"/>
</StackPanel>
```
Допускается в XAML заниматся форматированием отображаемого текста привязанного значения добавив свойство ContentStringFormat и передав ему специальную строку. Пример:
```csharp
<Label x:Name="labelThumb" Height="30" BorderBrush="Blue" BorderThickness="2" Content="{Binding Path=Value}" ContentStringFormat="Значение: {0:F0} едениц"/>
<Label x:Name="label2Thumb" Height="30" BorderBrush="Blue" BorderThickness="2" Content="{Binding Path=Value}" ContentStringFormat="{}{0:F0}"/>
```
Допускается в XAML не только форматировать, но еще преобразовывать данные между источником и целью. Для этого нужно создать класс, реализующий интерфейс IValueConverter. Такой класс можно применять для уточнения процесса привязки данных. Метод Convert() вызывается при передаче значения от источника к цели. Метод ConvertBack() будет вызываться, когда значение передается от цели к источнику - когда включен двунаправленный режим привязки.

Пример такого класса и добавления конвертирования значения с использованием рекурса:
```csharp
...
    WindowStartupLocation="CenterScreen">
<Window.Resources>
    <local:MyConverter x:Key="MyConverter"/>
</Window.Resources>
...
<Label x:Name="labelThumb" Height="30" BorderBrush="Blue" BorderThickness="2" Content="{Binding Path=Value,Converter={StaticResource MyConverter}}"/>
<Button Content="Нажми меня" Height="300" FontSize="{Binding Path=Value,Converter={StaticResource MyConverter}}"/>
class MyConverter : IValueConverter
{
    public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
    {
        return System.Convert.ToInt32((double)value) * 10;
    }
    public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
    {
        return value;
    }
}
```
Пример такой-же привязки, но с использованием кода C#:
```csharp
<Label x:Name="labelThumb" Height="30" BorderBrush="Blue" BorderThickness="2"/>
Binding b = new Binding
{
    Converter = new MyConverter(),
    Source = myScrollBar,
    Path = new PropertyPath("Value"),
};
labelThumb.SetBinding(Label.ContentProperty, b);
buttonOK.SetBinding(Button.FontSizeProperty, b);
```
Допускается привязывать данные из файлов XML, базы данных и объектов в памяти.

Пример привязки коллекции из памяти:
```csharp
<ListView x:Name="listViewPerson" ItemsSource="{Binding List}">
    <ListView.View>
        <GridView>
            <GridViewColumn Header="Имя" Width="100" DisplayMemberBinding="{Binding Name}"/>
            <GridViewColumn Header="Возраст" Width="80" DisplayMemberBinding="{Binding Age}"/>
        </GridView>
    </ListView.View>
</ListView>
public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
};
public MainWindow()
{
    InitializeComponent();
    List<Person> list = new List<Person>
    {
        new Person {Name = "Петя", Age = 23},
        new Person {Name = "Надя", Age = 18},
        new Person {Name = "Дима", Age = 20},
    };
    listViewPerson.ItemsSource = list;
```

## Свойства зависимости

Инфраструктура WPF поддерживает уникальную программную концепцию - свойство зависимости, выглядящие как обычные свойства, но за кулисами они реализуют специальные свойства для служб WPF, требующими взаимодействия через свойства зависимости. Также предназначены для инкапсуляции полей данных класса, могут быть настроены на только для чтения, только для записи, либо оба. Свойства Height,Width,Content на самом деле являются свойствами зависимости. Класс, который определяет свойство зависимости, должен иметь в своей цепочке наследования класс DependencyObject. Свойство зависимости представляется как открытое, статическое, только для чтения поле DependencyProperity. В классе будет определено дружественное к XAML свойство CLR, которое вызывает методы, предоставляемые классом DependencyObject, для получения и установки значения.

Никода не нужно помещать логику проверки достоверности в блок set и оболочка CLR свойства зависимости не должна делать ничего кроме вызовов GetValue() или SetValue().

- Свойства зависимости могут наследовать свои значения от определения XAML родительского элемента.

- Свойства зависимости поддерживают возможность получать значения, установленные элементами внутри их области определения XAML.

- Свойства зависимости дают WPF способность вычислять значения на основе множества внешних значений.

- Свойства зависимости дают поддержку инфраструктуры для триггеров WPF.

- Взаимодействовать с существующим свойством зависимости можно в манере, идентичной работе с обычным свойством CLR.

Если необходимо установить привязку данных в коде, нужно вызывать метод SetBinding():
```csharp
<ScrollBar x:Name="scrollBar" Orientation="Horizontal" Minimum="0" Maximum="100"></ScrollBar>
<Label x:Name="labelMyNumber" Content="0"></Label>
Binding b = new Binding
{
    Source = scrollBar,
    Path = new PropertyPath("Value"),
};
labelMyNumber.SetBinding(Label.ContentProperty, b);  
```

При регистрации свойства зависимости нужно использовать делегат ValidateValueCallbac для указания на метод, который выполняет проверку достоверности данных.







