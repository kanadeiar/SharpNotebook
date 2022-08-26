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
Еще пример привязки в разметке:
```csharp
<TextBox Name="txtValue" FontSize="20" />            
<WrapPanel Margin="0,10">
    <TextBlock Text="Текст: " FontSize="20" FontWeight="Bold" />
    <TextBlock x:Name="mirrorTextBlock"
               Text="{Binding ElementName=txtValue, Path=Text, Mode=OneWay}"   
               FontSize="20"/>
</WrapPanel>
```
Еще пример установки привязки в коде C#:
```csharp
<TextBox Name="txtValue" FontSize="20" />
<WrapPanel Margin="0,10">
    <TextBlock Text="Текст: " FontSize="20" FontWeight="Bold" />
    <TextBlock x:Name="mirrorTextBlock"
               Text="{Binding ElementName=txtValue, Path=Text, Mode=OneWay}"   
               FontSize="20"/>
</WrapPanel>
// Text = "{Binding ElementName=txtValue, Path=Text, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}"
Binding binding = new Binding
{
    ElementName = nameof(txtValue),
    //// элемент-источник
    Path = new PropertyPath(nameof(txtValue.Text)),
    Mode = BindingMode.TwoWay,
    UpdateSourceTrigger = UpdateSourceTrigger.PropertyChanged
};
mirrorTextBlock.SetBinding(TextBlock.TextProperty, binding);
```
Примеры простой привязки в C#:
```csharp
Binding bindingTitle = new Binding
{
    Source = _panel,
    Path = new PropertyPath("Title"),
};
this.SetBinding(TitleProperty, bindingTitle);
Binding bindingFam = new Binding
{
    Source = Employee,
    Path = new PropertyPath("Fam"),
};
TextBoxFamEmployee.SetBinding(TextBox.TextProperty, bindingFam);
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

Пример такого класса и добавления конвертирования значения с использованием ресурса:
```csharp
...
    WindowStartupLocation="CenterScreen">
<Window.Resources>
    <local:MyConverter x:Key="MyConverter"/>
</Window.Resources>
...
<Label x:Name="labelThumb" Height="30" BorderBrush="Blue" BorderThickness="2" Content="{Binding Path=Value,Converter={StaticResource MyConverter}}"/>
<Button Content="Нажми меня" Height="300" FontSize="{Binding Path=Value,Converter={StaticResource MyConverter}}"/>
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
Еще пример привязки к элементу, объявленному в разметке:
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
Пример привязки к объекту в памяти через установку контекста:
```csharp
class MyClass : INotifyPropertyChanged
{
    private bool flag;
    public bool Flag
    {
        get => flag;
        set
        {
            if (flag == value) return;
            flag = value;
            PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(nameof(Flag)));
        }
    }
    public ObservableCollection<int> Ints { get; set; }

    public MyClass()
    {
        flag = true;
        Ints = new ObservableCollection<int> {1, 2, 4, 8, 9};
    }
    public event PropertyChangedEventHandler PropertyChanged;
}
<Grid x:Name="GridMain">
    <StackPanel>
        <ListBox ItemsSource="{Binding Ints}" Height="200">
        </ListBox>
        <CheckBox Content="Выбери меня" IsChecked="{Binding Flag}"/>
        <RadioButton Content="Выбери меня" IsChecked="{Binding Flag}"/>
        <TextBlock Text="{Binding Flag}"/>
        <Button x:Name="ButtonEdit" Content="Изменить" Click="ButtonEdit_OnClick"/>
        <Button x:Name="ButtonAdd" Content="Добавить" Click="ButtonAdd_OnClick"/>
        <Button x:Name="ButtonEditInts" Content="Изменить массив" Click="ButtonEditInts_OnClick"/>
    </StackPanel>
</Grid>
public partial class MainWindow : Window
{
    private MyClass myClass;
    public MainWindow()
    {
        InitializeComponent();
        myClass = new MyClass();
        GridMain.DataContext = myClass; //установка контекста
    }
    private void ButtonEdit_OnClick(object sender, RoutedEventArgs e)
    {
        myClass.Flag = !myClass.Flag;
    }
    private void ButtonAdd_OnClick(object sender, RoutedEventArgs e)
    {
        myClass.Ints.Add(99);
    }
    private void ButtonEditInts_OnClick(object sender, RoutedEventArgs e)
    {
        myClass.Ints[0]++;
    }
}
```

Пример привязки к xml файлу:

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


Пример использования привязки к коллекции из родительского окна в дочернем окне:
```csharp
public class Person
{
    public string Fam { get; set; }
    public string Name { get; set; }
}
<StackPanel>
    <ListBox x:Name="ListBoxPersons" Height="400" MouseDoubleClick="ListBoxPersons_MouseDoubleClick">
        <ListBox.ItemTemplate>
            <DataTemplate>
                <StackPanel Orientation="Horizontal">
                    <TextBlock Text="{Binding Path=Fam}"/>
                    <TextBlock Text="{Binding Path=Name}"/>
                </StackPanel>
            </DataTemplate>
        </ListBox.ItemTemplate>
    </ListBox>
    <Button Height="40" Width="200" Content="Добавить" Click="ButtonAdd_Click"/>
</StackPanel>
public Person Select { get; set; }
ObservableCollection<Person> items;
public MainWindow()
{
    InitializeComponent();
    items = new ObservableCollection<Person>
    {
        new Person {Fam = "Иванов", Name = "Иван"},
        new Person {Fam = "Петров", Name = "Петр"},
    };
    ListBoxPersons.ItemsSource = items;
}
private void ButtonAdd_Click(object sender, RoutedEventArgs e)
{
    items.Add(new Person {Fam = "Тестов", Name = "Тест"});
}
private void ListBoxPersons_MouseDoubleClick(object sender, MouseButtonEventArgs e)
{
    Select = ListBoxPersons.SelectedItem as Person;
    DetailsWindow detailsWindow = new DetailsWindow(Select);
    detailsWindow.ShowDialog();
}
//дочернее окно
<StackPanel x:Name="StackPanelDetails">
    <TextBox Text="{Binding Path=Fam}" Margin="5"/>
    <TextBox Text="{Binding Path=Name}" Margin="5"/>
    <Button Content="OK" Margin="5" Click="Button_Click"/>
</StackPanel>
public DetailsWindow()
{
    InitializeComponent();
}
public DetailsWindow(Person person) : this()
{
    StackPanelDetails.DataContext = person;
}
private void Button_Click(object sender, RoutedEventArgs e)
{
    DialogResult = true;
}
```

Пример работы с двумя связанными по связи "один ко многим" моделями:
```csharp
//модели:
class Department : INotifyPropertyChanged
{
    #region Поля
    private string _name;
    public string Name
    {
        get => _name;
        set
        {
            if (value == _name) return;
            _name = value;
            PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(nameof(this.Name)));
        }
    }
    #endregion
    private ObservableCollection<Employee> _employees; //ссылка на всех сотрудников
    private ObservableCollection<Employee> _linkedEmployees;
    public ObservableCollection<Employee> LinkedEmployees
    {
        get
        {
            if (_linkedEmployees != null) return _linkedEmployees;
            _linkedEmployees = new ObservableCollection<Employee>();
            LinkedEmployeesUpdate();
            return _linkedEmployees;
        }
    } //сотрудники этого отдела
    public void LinkedEmployeesUpdate() //обновление сорудников отдела
    {
        var emp = _employees.Where(e => e.Department == this);
        LinkedEmployees.Clear();
        foreach (var el in emp)
            LinkedEmployees.Add(el);
    }
    public Department(string name)
    {
        Name = name;
    }
    public Department(string name, ObservableCollection<Employee> employees) : this (name)
    {
        //_linkedEmployees = new ObservableCollection<Employee>();
        _employees = employees;
    }
    public event PropertyChangedEventHandler PropertyChanged;
}
class Employee : INotifyPropertyChanged
{
    #region Поля
    private string _fam;
    public string Fam
    {
        get => _fam;
        set
        {
            if (value == _fam) return;
            _fam = value;
            PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(nameof(this.Fam)));
        }
    }
    private string _name;
    public string Name
    {
        get => _name;
        set
        {
            if (value == _name) return;
            _name = value;
            PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(nameof(this.Name)));
        }
    }
    private Department _department;
    public Department Department
    {
        get => _department;
        set
        {
            if (value == _department) return;
            Department old = _department;
            _department = value;
            old?.LinkedEmployeesUpdate(); //обновление сотрудников у отдела старого
            _department.LinkedEmployeesUpdate(); //обновление сотрудников у отдела нового
            PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(nameof(this.Department)));
        }
    }
    #endregion
    public Employee(string fam, string name)
    {
        Fam = fam;
        Name = name;
    }
    public Employee(string fam, string name, Department dep) : this(fam, name)
    {
        _department = dep;
    }
    public event PropertyChangedEventHandler PropertyChanged;
}
class Company
{
    public ObservableCollection<Department> Departments { get; set; }
    public ObservableCollection<Employee> Employees { get; set; }
    Random rnd = new Random();
    public Company(int countDeps, int countEmps)
    {
        Departments = new ObservableCollection<Department>();
        Employees = new ObservableCollection<Employee>();
        for (int i = 0; i < countDeps; i++)
            Departments.Add(new Department($"Отдел {i+1}", Employees));
        for (int i = 0; i < countEmps; i++)
            Employees.Add(new Employee($"Тестов {i+1}", $"Тест {i+1}", Departments[rnd.Next(countDeps)]));
        //foreach (var el in Departments)
        //    el.LinkedEmployeesUpdate();
    }
    public void AddEmployee(Employee newEmp)
    {
        if (newEmp == null) return;
        Employees.Add(newEmp);
        if (newEmp.Department is Department dep)
            dep.LinkedEmployeesUpdate();
    }
    public void DelEmployee(Employee delEmp)
    {
        if (delEmp == null) return;
        Employees.Remove(delEmp);
        if (delEmp.Department is Department dep)
            dep.LinkedEmployeesUpdate();
    }
}
//окно:
<Grid x:Name="GridMain">
    <Grid.ColumnDefinitions>
        <ColumnDefinition/>
        <ColumnDefinition Width="2*"/>
    </Grid.ColumnDefinitions>
    <Grid.RowDefinitions>
        <RowDefinition Height="3*"/>
        <RowDefinition/>
    </Grid.RowDefinitions>
    <ListBox x:Name="ListBoxDeps" Grid.Column="0" Grid.RowSpan="2" ItemsSource="{Binding Path=Departments}" Margin="5" VirtualizingStackPanel.IsVirtualizing="True">
        <ListBox.ItemsPanel>
            <ItemsPanelTemplate>
                <VirtualizingStackPanel/>
            </ItemsPanelTemplate>
        </ListBox.ItemsPanel>
        <ListBox.ItemTemplate>
            <DataTemplate>
                <TextBlock Text="{Binding Path=Name}"/>
            </DataTemplate>
        </ListBox.ItemTemplate>
    </ListBox>
    <ListBox x:Name="ListBoxEmps" Grid.Column="1" ItemsSource="{Binding ElementName=ListBoxDeps, Path=SelectedValue.LinkedEmployees}" Margin="5" VirtualizingStackPanel.IsVirtualizing="True">
        <ListBox.ItemsPanel>
            <ItemsPanelTemplate>
                <VirtualizingStackPanel/>
            </ItemsPanelTemplate>
        </ListBox.ItemsPanel>
        <ListBox.ItemTemplate>
            <DataTemplate>
                <TextBlock>
                    <TextBlock.Text>
                        <MultiBinding StringFormat=" {0} | {1}">
                            <Binding Path="Fam"/>
                            <Binding Path="Name"/>
                        </MultiBinding>
                    </TextBlock.Text>
                </TextBlock>
            </DataTemplate>
        </ListBox.ItemTemplate>
    </ListBox>
    <WrapPanel Grid.Row="1" Grid.Column="1">
        <StackPanel Orientation="Horizontal" Margin="3">
            <TextBlock Text="Фамилия: "/>
            <TextBox Text="{Binding ElementName=ListBoxEmps, Path=SelectedValue.Fam, UpdateSourceTrigger=PropertyChanged}" Width="100"/>
        </StackPanel>
        <StackPanel Orientation="Horizontal" Margin="3">
            <TextBlock Text=" Имя: "/>
            <TextBox Text="{Binding ElementName=ListBoxEmps, Path=SelectedValue.Name, UpdateSourceTrigger=PropertyChanged}" Width="100"/>
        </StackPanel>
        <StackPanel Orientation="Horizontal" Margin="3">
            <TextBlock Text=" Отдел: "/>
            <ComboBox ItemsSource="{Binding Path=Departments}" DisplayMemberPath="Name" SelectedItem="{Binding ElementName=ListBoxEmps, Path=SelectedValue.Department}" Width="100" VirtualizingStackPanel.IsVirtualizing="True">
                <ComboBox.ItemsPanel>
                    <ItemsPanelTemplate>
                        <VirtualizingStackPanel/>
                    </ItemsPanelTemplate>
                </ComboBox.ItemsPanel>
            </ComboBox>
        </StackPanel>
        <Button x:Name="ButtonAddEmp" Content="Добавить тестового" Margin="3"/>
        <Button x:Name="ButtonDelEmp" Content="Удалить выбранного" Margin="3"/>
    </WrapPanel>
</Grid>
private void Window_Loaded(object sender, RoutedEventArgs e)
{
    Company company = new Company(1000, 100000);
    GridMain.DataContext = company;
    ButtonAddEmp.Click += delegate 
    {
        company.AddEmployee(new Employee("Тестов", "Тест", (Department)ListBoxDeps.SelectedValue));
    };
    ButtonDelEmp.Click += delegate
    {
        company.DelEmployee((Employee)ListBoxEmps.SelectedValue);
    };
}
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








