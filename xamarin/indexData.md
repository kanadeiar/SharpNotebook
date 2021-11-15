# Данные Xamarin.Forms

## Данные

Вывод данных:
```csharp
public partial class MainPage : ContentPage
{
    public MainPage()
    {
        Label header = new Label
        {
            Text = "Список моделей",
            FontSize = Device.GetNamedSize(NamedSize.Large, typeof(Label)),
            HorizontalOptions = LayoutOptions.Center,
        };
        string[] phones = new string[] { "iPhone 7", "Samsung Galaxy S5", "Huawei P10", "LG G5" };
        var listView = new ListView();
        listView.ItemsSource = phones;
        this.Content = new StackLayout { Children = { header, listView } };
    }
}
```

Только Xaml:
```xml
<ContentPage.Resources>
    <ResourceDictionary>
        <x:Array x:Key="phones" Type="{x:Type x:String}">
            <x:String>iPhone 7</x:String>
            <x:String>Samsung Galaxy S5</x:String>
            <x:String>Huawei P10</x:String>
            <x:String>LG G5</x:String>
        </x:Array>
    </ResourceDictionary>
</ContentPage.Resources>
<StackLayout>
    <Label Text="Список моделей" FontSize="Large" HorizontalOptions="Center" />
    <ListView x:Name="ListViewPhones" ItemsSource="{StaticResource phones}"/>
</StackLayout>
```

Смешанный вид:
```xml
<Label Text="Список моделей" FontSize="Large" HorizontalOptions="Center" />
<ListView x:Name="ListViewPhones" ItemsSource="{Binding Phones}"/>
```
Код:
```csharp
public partial class MainPage : ContentPage
{
    public string[] Phones { get; set; }
    public MainPage()
    {
        InitializeComponent();
        Phones = new string[] { "iPhone 7", "Samsung Galaxy S5", "Huawei P10", "LG G5" };
        this.BindingContext = this;
    }
}
```

Привязка к выбранному элементу:
```xml
<Label Text="Выбранный элемент:" FontSize="Medium" />
<Label Text="{Binding Source={x:Reference ListViewPhones}, Path=SelectedItem}" FontSize="Large" FontAttributes="Bold" />
<Label Text="Список моделей:" FontSize="Medium" />
<ListView x:Name="ListViewPhones" ItemsSource="{...}"/>
```

Отмена выделения элемента:
```csharp
<ListView x:Name="ListViewPhones" ItemsSource="{...}" ItemSelected="ListViewPhones_ItemSelected"/>
private void ListViewPhones_ItemSelected(object sender, SelectedItemChangedEventArgs e)
{
    ((ListView)sender).SelectedItem = null;
}
```

## Сложные элементы в ListView

```xml
<Label Text="{Binding Source={x:Reference ListViewPhones}, Path=SelectedItem.Title}" FontSize="Large"/>
<ListView x:Name="ListViewPhones" 
            HasUnevenRows="True"
            ItemsSource="{Binding Phones}"
            ItemTapped="phonesList_ItemTapped">
    <ListView.ItemTemplate>
        <DataTemplate>
            <ViewCell>
                <ViewCell.View>
                    <StackLayout>
                        <Label Text="{Binding Company}" FontSize="Subtitle"/>
                        <StackLayout Orientation="Horizontal">
                            <Label Text="{Binding Title, StringFormat='Модель: {0}'}"/>
                            <Label Text="{Binding Price, StringFormat='Цена: {0} руб.'}"/>
                        </StackLayout>
                    </StackLayout>
                </ViewCell.View>
            </ViewCell>
        </DataTemplate>
    </ListView.ItemTemplate>            
</ListView>
```
Код:
```csharp
public class Phone
{
    public string Title { get; set; }
    public string Company { get; set; }
    public int Price { get; set; }
}
public partial class MainPage : ContentPage
{
    public List<Phone> Phones { get; set; }
    public MainPage()
    {
        InitializeComponent();
        Phones = new List<Phone>
        {
            new Phone {Title = "iPhone 7", Company = "Apple", Price = 80000 },
            new Phone {Title = "Samsung", Company = "Galaxy S5", Price = 12000 },
            new Phone {Title = "LG", Company = "G5", Price = 8000 },
            new Phone {Title = "Huawei", Company = "P10", Price = 20000 },
            new Phone {Title = "HTC", Company = "U Ultra", Price = 15000 },
        };
        this.BindingContext = this;
    }
    private async void phonesList_ItemTapped(object sender, ItemTappedEventArgs e)
    {
        var selected = e.Item as Phone;
        if (selected != null)
            await DisplayAlert("Выбранная модель", $"{selected.Company} - {selected.Title}", "OK");
    }
}
```

Использование элемента TextCell

```xml
<ListView.ItemTemplate>
    <DataTemplate>
        <TextCell 
            Text="{Binding Title}" 
            Detail="{Binding Company, StringFormat='Компания {0}'}" />
    </DataTemplate>
</ListView.ItemTemplate>
```

## Изображения в списках

```xml
<StackLayout Padding="5">
    <Label Text="{Binding Source={x:Reference ListViewPhones}, Path=SelectedItem.Title}" FontSize="Large"/>
    <ListView x:Name="ListViewPhones" 
                HasUnevenRows="True"
                ItemsSource="{Binding Phones}">
        <ListView.ItemTemplate>
            <DataTemplate x:DataType="local:Phone">
                <ImageCell 
                    ImageSource="{Binding ImagePath}"
                    Text="{Binding Title}"
                    Detail="{Binding Company, StringFormat='Модель {0}'}"
                    TextColor="Red"
                    DetailColor="Green"/>
            </DataTemplate>
        </ListView.ItemTemplate>
    </ListView>
</StackLayout>
```
Код:
```csharp
public class Phone
{
    public string Title { get; set; }
    public string Company { get; set; }
    public int Price { get; set; }
    public string ImagePath { get; set; }
}
public partial class MainPage : ContentPage
{
    public List<Phone> Phones { get; set; }
    public MainPage()
    {
        InitializeComponent();
        Phones = new List<Phone>
        {
            new Phone {Title = "iPhone 7", Company = "Apple", Price = 80000, ImagePath = "phone1.png" },
            new Phone {Title = "Samsung", Company = "Galaxy S5", Price = 12000, ImagePath = "phone2.png" },
            new Phone {Title = "LG", Company = "G5", Price = 8000, ImagePath = "phone3.png" },
            new Phone {Title = "Huawei", Company = "P10", Price = 20000, ImagePath = "phone1.png" },
            new Phone {Title = "HTC", Company = "U Ultra", Price = 15000, ImagePath = "phone2.png" },
            new Phone {Title = "Philips", Company = "S400", Price = 8000, ImagePath = "phone3.png" },
        };
        this.BindingContext = this;
    }
}
```

## Наблюдаемая коллекция

За счет реализации интерфейса INotifyCollectionChanged при добавлении или удалении объектов в ObservableCollection автоматически будут изменяться все привязанные к этой коллекции объекты, в том числе и ListView.

```xml
<StackLayout Padding="5">
    <Label Text="{Binding Source={x:Reference ListViewPhones}, Path=SelectedItem.Title}" FontSize="Large"/>
    <ListView x:Name="ListViewPhones" 
                HasUnevenRows="True"
                ItemsSource="{Binding Phones}">
        <ListView.ItemTemplate>
            <DataTemplate x:DataType="local:Phone">
                <ViewCell>
                    <ViewCell.View>
                        <StackLayout>
                            <Label Text="{Binding Title}" FontSize="Large" />
                            <Label Text="{Binding Company}" />
                            <Label Text="{Binding Price, StringFormat='Цена: {0} руб.'}" />
                        </StackLayout>
                    </ViewCell.View>
                </ViewCell>
            </DataTemplate>
        </ListView.ItemTemplate>
    </ListView>
    <StackLayout Orientation="Horizontal" HorizontalOptions="Center">
        <Button Text="Добавить" Clicked="ButtonAdd_Clicked" />
        <Button Text="Удалить" Clicked="ButtonDelete_Clicked" />
    </StackLayout>
</StackLayout>
```
Код:
```csharp
public class Phone
{
    public string Title { get; set; }
    public string Company { get; set; }
    public int Price { get; set; }
}
public partial class MainPage : ContentPage
{
    public ObservableCollection<Phone> Phones { get; set; }
    public MainPage()
    {
        InitializeComponent();
        Phones = new ObservableCollection<Phone>
        {
            new Phone {Title = "iPhone 7", Company = "Apple", Price = 80000 },
            new Phone {Title = "Samsung", Company = "Galaxy S5", Price = 12000 },
            new Phone {Title = "LG", Company = "G5", Price = 8000 },
            new Phone {Title = "Huawei", Company = "P10", Price = 20000 },
            new Phone {Title = "HTC", Company = "U Ultra", Price = 15000 },
            new Phone {Title = "Philips", Company = "S400", Price = 8000 },
        };
        this.BindingContext = this;
    }

    private void ButtonAdd_Clicked(object sender, EventArgs e)
    {
        Phones.Add(new Phone { Title = "Newest", Company = "Test", Price = 999 });
    }

    private void ButtonDelete_Clicked(object sender, EventArgs e)
    {
        var phone = ListViewPhones.SelectedItem as Phone;
        if (phone != null)
        {
            Phones.Remove(phone);
            ListViewPhones.SelectedItem = null;
        }
    }
}
```