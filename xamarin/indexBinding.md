# Привязки Xamarin.Forms

## Основы привязок

Для поддержки привязки данных Xamarin Forms определяет класс BindableObject. Отличительной особенностью класса BindableObject является то, что он содержит специальные типы свойств BindableProperty. обычные свойства по сути представляют обертку над BindableProperty. В .NET есть похожая концепция - свойства зависимости (dependency property), которые имеют похожее назначение.

Как и в WPF приложениях, здесь классы, доступные для привязок должны реализовывать INotifyPropertyChanged. 

Режим привязки определяет в какую сторону буде работать привязка:

- Default: значение по умолчанию. Для разных элементов и свойств может отличаться

- OneWay: при изменении в источнике изменяется цель

- OneTime: привязка установливается только однократно и больше не действует. То есть цель привязки получает начальное значение из источника привязки. Но при дальнейшем изменении в источнике привязка больше не действует

- OneWayToSource: при изменении в цели привязки изменяется источник (то есть обратная привязка)

- TwoWay: изменения в источнике воздействуют на цель привязки, а изменении в цели воздействуют на источник (то есть двусторонняя привязка)

Простая привязка в коде C#:

```csharp
public BindPage()
{
    var label = new Label
    {
        FontSize = Device.GetNamedSize(NamedSize.Large, typeof(Label))
    };
    var entry = new Entry();
    label.BindingContext = entry;
    label.SetBinding(Label.TextProperty, "Text");
    var stackLayout = new StackLayout()
    {
        Children = { label, entry }
    };
    Content = stackLayout;
    //InitializeComponent();
}
```

Привязка в Xaml:
```xml
<Label x:Name="LabelBox" FontSize="Large" BindingContext="{x:Reference EntryBox}" Text="{Binding Text}" />
<Entry x:Name="EntryBox" />
```

Еще пример привязки:
```xml
<Label x:Name="label" FontSize="Large" Text="{Binding Source={x:Reference Name=entry}, Path=Text}" />
<Entry x:Name="entry" />
```

Свойство StringFormat класса Binding позволяет дополнительно отформатировать значение.

Пример использования:
```xml
<Label FontSize="Large" Text="{Binding Source={x:Reference SliderOne}, Path=Value, StringFormat='Установлено: {0}'}" />
<Slider x:Name="SliderOne" Minimum="0" Maximum="100" Value="50" MaximumTrackColor="LawnGreen" MinimumTrackColor="ForestGreen" />
```

## Конвертеры значений

Конвертор дат:
```csharp
public class DateTimeToLocalDateTimeConverter : IValueConverter
{
    public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
    {
        return ((DateTime)value).ToString("dd.MM.yyyy");
    }
    public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
    {
        return null;
    }
}
```

Конвертор цветов:
```csharp
public class StringToColorConverter : IValueConverter
{
    public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
    {
        var color = Color.White;
        var str = value.ToString().ToLower();
        switch (str)
        {
            case "красный": color = Color.Red; break;
            case "желтый": color = Color.Yellow; break;
            case "зеленый": color = Color.Green; break;
            case "синий": color = Color.Blue; break;
        }
        return color;
    }
    public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
    {
        return Color.White;
    }
}
```

Использование:
```xml
        xmlns:local="clr-namespace:HellpApp"
        ...>
<ContentPage.Resources>
    <ResourceDictionary>
        <local:DateTimeToLocalDateTimeConverter x:Key="dateConverter" />
        <local:StringToColorConverter x:Key="colorConvertor" />
    </ResourceDictionary>
</ContentPage.Resources>
<StackLayout>
    <Label FontSize="Large" Text="{Binding Source={x:Reference DatePicker}, Path=Date}" />
    <Label Text="{Binding Source={x:Reference DatePicker}, Path=Date, Converter={StaticResource dateConverter}}" />
    <DatePicker x:Name="DatePicker" Format="D" />
    <Label Text="Цвет:" />
    <Entry x:Name="EntryColor" Text="Red" />
    <Label x:Name="LabelColor" Text="Xamarin" TextColor="{Binding Source={x:Reference EntryColor}, Converter={StaticResource colorConvertor}, Path=Text}" />
</StackLayout>
```

## Привязка к объектам

Объект с привязками:
```csharp
public class Phone : INotifyPropertyChanged
{
    private string _title;
    public string Title
    {
        get => _title;
        set => Set(ref _title, value);
    }
    private string _company;
    public string Company
    {
        get => _company;
        set => Set(ref _company, value);
    }
    private int _price;
    public int Price
    {
        get => _price;
        set => Set(ref _price, value);
    }

    public event PropertyChangedEventHandler PropertyChanged;
    public void OnPropertyChanged([CallerMemberName] string propertyName = null)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
    }
    protected virtual bool Set<T>(ref T field, T value, [CallerMemberName] string propertyName = null)
    {
        if (Equals(field, value)) return false;
        field = value;
        OnPropertyChanged(propertyName);
        return true;
    }
}
```
xaml:
```xml
<ContentPage ...
             xmlns:local="clr-namespace:HellpApp"
             ...>
    <ContentPage.Resources>
        <ResourceDictionary>
            <local:Phone x:Key="phone" Title="iPhone 7" Company="Apple" Price="5000" />
        </ResourceDictionary>
    </ContentPage.Resources>
    <StackLayout>
        <Grid BindingContext="{StaticResource phone}">
            <Grid.ColumnDefinitions>
                <ColumnDefinition Width="0.8*" />
                <ColumnDefinition Width="1.1*" />
                <ColumnDefinition Width="1.1*" />
            </Grid.ColumnDefinitions>
            <Grid.RowDefinitions>
                <RowDefinition Height="50" />
                <RowDefinition Height="50" />
                <RowDefinition Height="50" />
            </Grid.RowDefinitions>
            <Label Grid.Column="0" Grid.Row="0" Text="Модель" />
            <Label Grid.Column="1" Grid.Row="0" Text="Компания" />
            <Label Grid.Column="2" Grid.Row="0" Text="Цена" />

            <Label Grid.Column="0" Grid.Row="1" Text="{Binding Title}" />
            <Label Grid.Column="1" Grid.Row="1" Text="{Binding Company}" />
            <Label Grid.Column="2" Grid.Row="1" Text="{Binding Price}" />

            <Button Text="Обновить" Grid.Column="2" Grid.Row="2" Clicked="Button_Clicked" />
        </Grid>
    </StackLayout>
</ContentPage>
```
Код обновления:
```csharp
private void Button_Clicked(object sender, EventArgs e)
{
    var phone = this.Resources["phone"] as Phone;
    phone.Title = "Test";
    phone.Company = "Amd";
    phone.Price += 4000;
}
```
