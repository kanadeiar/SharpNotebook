# Привязки

Привязка данных представляет собой действие по подключению свойств элемента управления к значениям данных, изменяющихся на протяжении жизненного цикла приложения. Источником операции привязки данных являются сами данные, а местом назначения - свойство элемента управления пользовательского интерфейса. Инфраструктура WPF дает развитую экосистему привязки данных, способную поддерживаться в разметке. И обеспечивает синхронизацию источника и цели в случае изменения значений данных.

Механизмом, обеспечивающим определение привязки в разметке XAML, является расширение разметки {Binding}. Привязки можно определять как посредством кода C#, так и прямо в разметке. Значение ElementName указывает источник операции привязки данных, а Path - свойство к которому осуществляется привязка.

Пример:

```csharp
<StackPanel>
    <Label Content="Переместить ползунок"/>
    <ScrollBar x:Name="MyScrollBar" Orientation="Horizontal" Height="30" Minimum="0" Maximum="100" LargeChange="1" SmallChange="1"/>
    <Label x:Name="LabelThumb" Height="30" BorderBrush="Blue" BorderThickness="2" Content="{Binding ElementName=MyScrollBar, Path=Value}"/>
</StackPanel>
```

## Контекст привязки

В XAML для определения привязки данных можно использовать альтернативный формат, разбивающий значения, указанные расширенной разметкой {Bunding}, путем установки свойства DataContext.

```csharp
<Label x:Name="labelThumb" Height="30" BorderBrush="Blue" BorderThickness="2" DataContext="{Binding ElementName=myScrollBar}" Content="{Binding Path=Value}"/>
```

Это может быть удобно из-за того, что подэлементы наследуют свои значения в дереве разметки.

Пример:

```csharp
<StackPanel DataContext="{Binding ElementName=myScrollBar}">
    <Label Content="Переместить ползунок:"/>
    <ScrollBar x:Name="myScrollBar" Orientation="Horizontal" Height="30" Minimum="1" Maximum="100" LargeChange="1" SmallChange="1"/>
    <Label x:Name="labelThumb" Height="30" BorderBrush="Blue" BorderThickness="2" Content="{Binding Path=Value}"/>
    <Button Content="Нажми меня" Height="300" FontSize="{Binding Path=Value}"/>
</StackPanel>
```

В XAML можно заниматся форматированием отображаемого текста привязанного значения добавив свойство ContentStringFormat и передав ему специальную строку. 

Пример:

```csharp
<Label x:Name="labelThumb" Height="30" BorderBrush="Blue" BorderThickness="2" Content="{Binding Path=Value}" ContentStringFormat="Значение: {0:F0} едениц"/>
<Label x:Name="label2Thumb" Height="30" BorderBrush="Blue" BorderThickness="2" Content="{Binding Path=Value}" ContentStringFormat="{}{0:F0}"/>
```

Допускается в XAML не только форматировать, но еще преобразовывать данные между источником и целью. Для этого нужно создать класс, реализующий интерфейс IValueConverter. Такой класс можно применять для уточнения процесса привязки данных. Метод Convert() вызывается при передаче значения от источника к цели. Метод ConvertBack() будет вызываться, когда значение передается от цели к источнику - когда включен двунаправленный режим привязки.

Пример:

```csharp
...
    WindowStartupLocation="CenterScreen">
<Window.Resources>
    <local:MyConverter x:Key="MyConverter"/>
</Window.Resources>
...
<Label x:Name="labelThumb" Height="30" BorderBrush="Blue" BorderThickness="2" Content="{Binding Path=Value,Converter={StaticResource MyConverter}}"/>
<Button Content="Нажми меня" Height="300" FontSize="{Binding Path=Value, Converter={StaticResource MyConverter}}"/>
[ValueConversion(typeof(double),typeof(int))]
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

Пример привязки к элементу, объявленному в разметке:

```csharp
...
<Window.Resources>
    <local:Employee x:Key="worker" Name="Петя" Age="30" Salary="25000" />
</Window.Resources>
<Grid x:Name="mainGrid">
    <StackPanel Margin="10" >
        <TextBlock Text="{Binding Source={StaticResource worker}, Path=Salary}" Margin="5" FontSize="20"/>
        <TextBlock FontSize="20" Margin="5" Text="{Binding Source={StaticResource worker}, Path=Salary, StringFormat = Зарплата составляет {0} ₽}" />
        <TextBlock x:Name="txtView" Margin="5" FontSize="20" Text="{Binding Source={StaticResource worker}, Path=Name}"/>
```

Допускается привязывать данные из файлов XML, базы данных и объектов в памяти.

### Пример привязки к xml файлу

Файл "MyXMLFile.xml":

```csharp
<?xml version="1.0" encoding="utf-8" ?>
<persons>
  <person>
    <id>1</id>
    <fam>Иванов</fam>
    <name>Иван</name>
    <age>20</age>
  </person>
  <person>
    <id>2</id>
    <fam>Петров</fam>
    <name>Петя</name>
    <age>18</age>
  </person>
  <person>
    <id>3</id>
    <fam>Генадиев</fam>
    <name>Гена</name>
    <age>30</age>
  </person>
</persons>
```

Код xaml:

```csharp
<Window.Resources>
    <XmlDataProvider x:Key="personsXMLData" Source="MyXMLFile.xml" XPath="persons"/>
</Window.Resources>
<Grid>
    <ListBox ItemsSource="{Binding Source={StaticResource personsXMLData}, XPath=person}">
        <ListBox.ItemTemplate>
            <DataTemplate>
                <StackPanel Orientation="Horizontal">
                    <TextBlock Text="{Binding XPath=fam}" Margin="0,0,5,0"/>
                    <TextBlock Text="{Binding XPath=name}" Margin="0,0,5,0"/>
                    <TextBlock Text="{Binding XPath=age}" Margin="0,0,5,0"/>
                    <TextBlock Text="лет"/>
                </StackPanel>
            </DataTemplate>
        </ListBox.ItemTemplate>
    </ListBox>
</Grid>
```

### Пример привязки коллекции из памяти

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




