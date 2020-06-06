# WPF MVVM Уведомления

Система привязки, встроенная в приложения на основе XAML позволяет привязывать объекты данных и коллекции к системе уведомлений, разрабатывая их как наблюдаемые. При изменениях в наблюдаемой модели либо в наблюдаемой коллекции, инидицируется событие - NotifyPropertyChanged или NotifyCollectionChanged. Инфтраструктура привязки прослушивает такие события и пи их появлении обновляет призязанные элементы управления.

Конструкции привязки данных вращаются вокруг контекста данных, который может быть установлен в самом элементе управления или в родительском элементе управления. Все содержащиеся внутри него элементы управления унаследуют результирующий контекст данных.

Чтобы изменения в коде отражались в интерфейсе нужно преобразовать класс-модель в наблюдаемую модель, использованием интерфейса INotifyPropertyChanged, который содержит единственное событие - PropertyChangedEvent. Механизм привязки XAML прослушивает это событие для каждого привязанного свойства в классах, реализующих этот интерфейс.

Обновление пользовательского интерфейса при изменении содержимого коллекции достигается путем реализации интерфейса INotifyCollectionChanged, этот интерфейс также открывает событие CollectionChanged. Это достигается примененением класса ObservableCollection<T>, который реализует этот интерфейс.

Свойство IsSharedSizeScope элемента управления Grid заставляет дочерние сетики разделять размеры. Элементы с пометкой в свойстве SharedSizeGroup получают туже ширину без доппрограммирования.

Пример реализации наблюдаемой модели и наблюдаемой коллекции первоначальная:
```csharp
...
<Grid IsSharedSizeScope="True" Margin="5">
    <Grid.RowDefinitions>
        <RowDefinition Height="Auto"/>
        <RowDefinition Height="Auto"/>
    </Grid.RowDefinitions>
    <Grid Grid.Row="0">
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="Auto" SharedSizeGroup="CarLabels"/>
            <ColumnDefinition Width="*"/>
        </Grid.ColumnDefinitions>
        <Label Grid.Column="0" Content="Автомобиль"/>
        <ComboBox Name="comboBoxCars" Grid.Column="1" DisplayMemberPath="PetName"/>
    </Grid>
    <Grid Grid.Row="1" Name="DetailsGrid" DataContext="{Binding ElementName=comboBoxCars, Path=SelectedItem}">
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="Auto" SharedSizeGroup="CarLabels"/>
            <ColumnDefinition Width="*"/>
        </Grid.ColumnDefinitions>
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
        </Grid.RowDefinitions>
        <Label Grid.Column="0" Grid.Row="0" Content="Марка" Margin="3"/>
        <TextBox Grid.Column="1" Grid.Row="0" Margin="3" Text="{Binding Path=Make}"/>
        <Label Grid.Column="0" Grid.Row="1" Content="Цвет" Margin="3"/>
        <TextBox Grid.Column="1" Grid.Row="1" Margin="3" Text="{Binding Path=Color}"/>
        <Label Grid.Column="0" Grid.Row="2" Margin="3" Content="Название"/>
        <TextBox Grid.Column="1" Grid.Row="2" Margin="3" Text="{Binding Path=PetName}"/>
        <StackPanel Grid.Column="0" Grid.ColumnSpan="2" Grid.Row="3" Orientation="Horizontal" HorizontalAlignment="Right">
            <Button x:Name="buttonAddCar" Content="Добавить автомобиль" Margin="5" Padding="4, 2" Click="buttonAddCar_Click"/>
            <Button x:Name="buttonChangeColor" Click="ButtonChangeColor_OnClick" Content="Изменить цвет" Margin="5" Padding="4, 2"/>
        </StackPanel>
    </Grid>
</Grid>
public partial class MainWindow : Window
{
    readonly IList<Inventory> cars = new ObservableCollection<Inventory>(); //наблюдаемая коллекция
    public MainWindow()
    {
        InitializeComponent();
        cars.Add(new Inventory
        {
            CarId = 1,
            Make = "УАЗ",
            Color = "Белый",
            PetName = "Буханка",
        });
        cars.Add(new Inventory
        {
            CarId = 2,
            Make = "ЗАЗ",
            Color = "Коричневый",
            PetName = "Большая"
        });
        comboBoxCars.ItemsSource = cars;
    }
    private void ButtonChangeColor_OnClick(object sender, RoutedEventArgs e)
    {
        cars.First(x => x.CarId == ((Inventory) comboBoxCars.SelectedItem)?.CarId).Color = "Pink";
    }
    private void buttonAddCar_Click(object sender, RoutedEventArgs e)
    {
        var maxCount = cars?.Max(x => x.CarId) ?? 0;
        cars.Add(new Inventory
        {
            CarId = maxCount + 1,
            Color = "Желтый",
            Make = "ВАЗ",
            PetName = "Копейка",
        });
    }
}
class Inventory : INotifyPropertyChanged //наблюдаемая модель
{
    private int carId;
    public int CarId
    {
        get => carId;
        set
        {
            if (value == carId) return;
            carId = value;
            //передача названия свойства значение которого изменилось
            OnPropertyChanged(nameof(CarId));
        }
    }
    private string make;
    public string Make
    {
        get => make;
        set
        {
            if (value == make) return;
            make = value;
            OnPropertyChanged(nameof(Make));
        }
    }
    private string color;
    public string Color
    {
        get => color;
        set
        {
            if (value == color) return;
            color = value;
            OnPropertyChanged(nameof(Color));
        }
    }
    private string petName;
    public string PetName
    {
        get => petName;
        set
        {
            if (value == petName) return;
            petName = value;
            OnPropertyChanged(nameof(PetName));
        }
    }
    public event PropertyChangedEventHandler PropertyChanged;
    /// <summary> Вспомогательный метод индицирования события изменения свойства </summary>
    protected void OnPropertyChanged([CallerMemberName] string propertyName = "")
    {
        //индицирование события со строкой, указывающей свойство, которое было изменено и нуждается в обновлении
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
        //обновление всех привязанных свойств объекта
        //PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(string.Empty));
    }
}
```
Пример с добавлением флага изменения:
```csharp
<Grid Grid.Row="1" Name="DetailsGrid" DataContext="{Binding ElementName=comboBoxCars, Path=SelectedItem}">
...
    <Label Grid.Column="0" Grid.Row="5" Content="Изменено"/>
    <CheckBox Grid.Column="1" Grid.Row="5" VerticalAlignment="Center" IsEnabled="False" IsChecked="{Binding Path=IsChanged}"/>
...
public partial class MainWindow : Window
...
    cars.Add(new Inventory
    {
        CarId = 1,
        Make = "УАЗ",
        Color = "Белый",
        PetName = "Буханка",
        IsChanged = false,
    });
    cars.Add(new Inventory
    {
        CarId = 2,
        Make = "ЗАЗ",
        Color = "Коричневый",
        PetName = "Большая",
        IsChanged = false,
    });
    comboBoxCars.ItemsSource = cars;
...
class Inventory : INotifyPropertyChanged //наблюдаемая модель
{
...
    //флаг изменения модели
    private bool isChanged;
    public bool IsChanged
    {
        get => isChanged;
        set
        {
            if (value == isChanged) return;
            isChanged = value;
            OnPropertyChanged(nameof(IsChanged));
        }
    }
    ...
    /// <summary> Вспомогательный метод индицирования события изменения свойства </summary>
    protected void OnPropertyChanged([CallerMemberName] string propertyName = "")
    {
        if (propertyName != nameof(IsChanged))
        {
            IsChanged = true;
        }
        //индицирование события со строкой, указывающей свойство, которое было изменено и нуждается в обновлении
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
        //обновление всех привязанных свойств объекта
        //PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(string.Empty));
    }
...
```
Проект с исходным кодом PropertyChanged.Fosy упрощает реализацию интерфейса InotifyPropertyChanged. Он добавляет весь связующий код для INotifyPropertyChanged, при наличии свойства IsChanged, оно будет обновляться, когда изменяется другое свойство. Необходимо добавить пакет PropertyChanged.Fody и установить его, а также добавить Fody. Пример класса-модели с использованием PropertyChanged.Fody:
```csharp
class Inventory : INotifyPropertyChanged
{
    public bool IsChanged { get; set; }
    public int CarId { get; set; }
    public string Make { get; set; }
    public string Color { get; set; }
    public string PetName { get; set; }
    public event PropertyChangedEventHandler PropertyChanged;
}
```



