# Навигация Xamarin.Forms

## Основы навигации

Пример навигации:

App.xaml.cs
```csharp
public App()
{
    InitializeComponent();
    MainPage = new NavigationPage(new MainPage());
}
public class CommonPage : ContentPage
{
    public CommonPage()
    {
        Title = "Common Page";
        Button backButton = new Button
        {
            Text = "Назад",
            HorizontalOptions = LayoutOptions.Center,
            VerticalOptions = LayoutOptions.Center,
        };
        backButton.Clicked += BackButton_Clicked;
        Content = backButton;
    }
    private async void BackButton_Clicked(object sender, EventArgs e)
    {
        await Navigation.PopAsync();
    }
}
public class ModalPage : ContentPage
{
    public ModalPage()
    {
        Title = "Modal Page";
        Button backButton = new Button
        {
            Text = "Назад",
            HorizontalOptions = LayoutOptions.Center,
            VerticalOptions = LayoutOptions.Center,
        };
        backButton.Clicked += BackButton_Clicked;
        Content = backButton;
    }
    private async void BackButton_Clicked(object sender, EventArgs e)
    {
        await Navigation.PopModalAsync();
    }
}
<StackLayout Padding="5">
    <Button x:Name="ToCommonPage" Text="На обычную страницу" HorizontalOptions="Center" VerticalOptions="CenterAndExpand" Clicked="ToCommonPage_Clicked" />
    <Button x:Name="ToModalPage" Text="На модальную страницу" HorizontalOptions="Center" VerticalOptions="CenterAndExpand" Clicked="ToModalPage_Clicked" />
</StackLayout>
private async void ToCommonPage_Clicked(object sender, EventArgs e)
{
    await Navigation.PushAsync(new CommonPage());
}
private async void ToModalPage_Clicked(object sender, EventArgs e)
{
    await Navigation.PushModalAsync(new ModalPage());
}
```

## Передача данных при навигации

Класс сущности:

```csharp
public class Phone : INotifyPropertyChanged
{
    private string _name;
    public string Name
    {
        get => _name;
        set => Set(ref _name, value);
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
    protected virtual void OnPropertyChanged([CallerMemberName] string propertyName = null)
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

Главная страница:

```xml
<StackLayout Padding="5">
    <Button Text="Добавить" Clicked="ButtonAdd_OnClicked"></Button>
    <ListView x:Name="ListViewPhones" ItemsSource="{Binding}" ItemSelected="ListViewPhones_OnItemSelected" >
        <ListView.ItemTemplate>
            <DataTemplate x:DataType="local:Phone">
                <ViewCell>
                    <ViewCell.View>
                        <Grid>
                            <Grid.ColumnDefinitions>
                                <ColumnDefinition Width="*" />
                                <ColumnDefinition Width="*" />
                                <ColumnDefinition Width="*" />
                            </Grid.ColumnDefinitions>
                            <Label Grid.Column="0" Text="{Binding Name}"></Label>
                            <Label Grid.Column="1" Text="{Binding Company}"></Label>
                            <Label Grid.Column="2" Text="{Binding Price}"></Label>
                        </Grid>
                    </ViewCell.View>
                </ViewCell>
            </DataTemplate>
        </ListView.ItemTemplate>
    </ListView>
</StackLayout>
```

```csharp
public partial class MainPage : ContentPage
{
    protected internal ObservableCollection<Phone> Phones { get; set; }
    public MainPage()
    {
        InitializeComponent();
        Phones = new ObservableCollection<Phone>
        {
            new Phone { Name = "iPhone 7", Company = "Apple", Price = 52000 },
            new Phone { Name = "Galaxy 9", Company = "Samsung", Price = 88000 },
            new Phone { Name = "G5", Company = "LG", Price = 34000 },
            new Phone { Name = "Honor 10", Company = "Heawei", Price = 99000 },
        };
        ListViewPhones.BindingContext = Phones;
    }
    private async void ButtonAdd_OnClicked(object sender, EventArgs e)
    {
        await Navigation.PushAsync(new PhonePage(null));
    }
    private async void ListViewPhones_OnItemSelected(object sender, SelectedItemChangedEventArgs e)
    {
        if (e.SelectedItem is Phone selected)
        {
            ListViewPhones.SelectedItem = null;
            await Navigation.PushAsync(new PhonePage(selected));
        }
    }
    protected internal void AddPhone(Phone phone)
    {
        Phones.Add(phone);
    }
}
```

Вторичная страница:

```xml
<StackLayout Padding="5" x:DataType="local:Phone">
    <Label Text="Информация"></Label>
    <Entry x:Name="EntryName" Text="{Binding Name}"></Entry>
    <Entry x:Name="EntryCompany" Text="{Binding Company}"></Entry>
    <StackLayout Orientation="Horizontal">
        <Stepper x:Name="StepperPrice" Minimum="0" Maximum="10000" Increment="100" Value="{Binding Price}"></Stepper>
        <Label x:Name="LabelPrice" FontSize="Large" Text="{Binding Source={x:Reference StepperPrice}, Path=Value}"></Label>
    </StackLayout>
    <Button Text="Сохранить" Clicked="SaveButton_OnClicked"></Button>
</StackLayout>
```

```csharp
public partial class PhonePage : ContentPage
{
    private bool edited = true;
    public Phone Phone { get; set; }
    public PhonePage(Phone phone)
    {
        InitializeComponent();
        Phone = phone;
        if (phone == null)
        {
            Phone = new Phone();
            edited = false;
        }
        this.BindingContext = Phone;
    }

    private async void SaveButton_OnClicked(object sender, EventArgs e)
    {
        await Navigation.PopAsync();

        if (edited == false)
        {
            NavigationPage navPage = (NavigationPage)Application.Current.MainPage;
            IReadOnlyList<Page> navStack = navPage.Navigation.NavigationStack;
            MainPage homePage = navStack[navPage.Navigation.NavigationStack.Count - 1] as MainPage;

            if (homePage != null)
            {
                homePage.AddPhone(Phone);
            }
        }
    }
}
```
