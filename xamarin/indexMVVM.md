# MVVM Xamarin.Forms

## Основы

Паттерн MVVM (Model - View - ViewModel) основывается на разделении функциональной части приложения на три ключевых компонента:

- View - представление или пользовательский интерфейс

- Model - модель или данные, которые используются в приложении

- ViewModel - промежуточный слой между представлением и данными, который обеспечивает их взаимодействие

```csharp
public class Phone
{
    public string Name { get; set; }
    public string Company { get; set; }
    public int Price { get; set; }
}
```

```csharp
public class PhoneViewModel : INotifyPropertyChanged
{
    private Phone _phone;
    public PhoneViewModel()
    {
        _phone = new Phone();
        Name = "iPhone 7";
        Company = "Apple";
        Price = 999;
    }
    public string Name
    {
        get => _phone.Name;
        set
        {
            if (Equals(_phone.Name, value)) return;
            _phone.Name = value;
            OnPropertyChanged(nameof(Name));
        }
    }
    public string Company
    {
        get => _phone.Company;
        set
        {
            if (Equals(_phone.Company, value)) return;
            _phone.Company = value;
            OnPropertyChanged(nameof(Company));
        }
    }
    public int Price
    {
        get => _phone.Price;
        set
        {
            if (Equals(_phone.Price, value)) return;
            _phone.Price = value;
            OnPropertyChanged(nameof(Price));
        }
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

```xml
<ContentPage xmlns="http://xamarin.com/schemas/2014/forms"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="HellpApp.MainPage"
             Title="Приложение"
             BindingContext="{Binding Source={StaticResource PhoneViewModel}}">
    <StackLayout Padding="5">
        <Label Text="{Binding Name}" FontSize="Large" />
        <Label Text="{Binding Company}" FontSize="Large" />
        <Label Text="{Binding Price, StringFormat='Цена: {0} руб.'}" FontSize="Large" />
    </StackLayout>
</ContentPage>
```

## Команды

В платформе Xamarin Forms имеется механизм команд.

```csharp
public interface ICommand 
{ 
    void Execute(object arg); 
    bool CanExecute(object arg); 
    event EventHandler CanExecuteChanged; 
}
```

Встроенную поддержку команд имеют следующие элементы управления:

- Button

- MenuItem

- ToolbarItem

- SearchBar

- TextCell

- ImageCell

- ListView

- TapGestureRecognizer

В Xamarin.Forms есть встроенный класс Command, который реализует всю функциональность. Нам же достаточно передать в его конструктор нужно действие, которое соответствует делегату Action.

```xml
<Button Text="+" Command="{Binding IncreaseCommand}" />
```

Во вьюмодели:
```csharp
private ICommand _IncreaseCommand;
/// <summary> Увеличение стоимости </summary>
public ICommand IncreaseCommand => _IncreaseCommand ?? (_IncreaseCommand = new Command(OnIncreaseCommandExecuted, CanIncreaseCommandExecute));
private bool CanIncreaseCommandExecute(object p) => true;
private void OnIncreaseCommandExecuted(object p)
{
    if (_phone != null)
        Price += 100;
}
```

## Пример приложения MVVM

- Папка Models

- Папка ViewModels

- Папка Views

```csharp
public class Friend
{
    public string Name { get; set; }
    public string Email { get; set; }
    public string Phone { get; set; }
}
```
App.xaml.cs
```csharp
MainPage = new NavigationPage(new FriendsListPage());
```
Вебмодели
```csharp
public class FriendsListViewModel : INotifyPropertyChanged
{
    private readonly INavigation _navigation;
    public ObservableCollection<FriendViewModel> Friends { get; set; }
        
    public FriendsListViewModel(INavigation navigation)
    {
        _navigation = navigation;
        Friends = new ObservableCollection<FriendViewModel>();
    }

    private FriendViewModel _selectedFriend;
    public FriendViewModel SelectedFriend
    {
        get => _selectedFriend;
        set
        {
            if (Equals(_selectedFriend, value)) return;
            var tempFriend = value;
            OnPropertyChanged(nameof(SelectedFriend));
            _navigation.PushAsync(new FriendPage(tempFriend));
        }
    }

    private ICommand _CreateFriendCommand;
    /// <summary> Создание друга </summary>
    public ICommand CreateFriendCommand => _CreateFriendCommand ??= new Command(OnCreateFriendCommandExecuted);
    private void OnCreateFriendCommandExecuted(object p)
    {
        _navigation.PushAsync(new FriendPage(new FriendViewModel() { ListViewModel = this }));
    }

    private ICommand _BackCommand;
    /// <summary> Назад </summary>
    public ICommand BackCommand => _BackCommand ??= new Command(OnBackCommandExecuted);
    private void OnBackCommandExecuted(object p)
    {
        _navigation.PopAsync();
    }

    private ICommand _SaveCommand;
    /// <summary> Сохранить друга </summary>
    public ICommand SaveCommand => _SaveCommand ??= new Command(OnSaveCommandExecuted);
    private void OnSaveCommandExecuted(object p)
    {
        var friend = p as FriendViewModel;
        if (friend != null && friend.IsValid && !Friends.Contains(friend))
        {
            Friends.Add(friend);
        }
        _navigation.PopAsync();
    }

    private ICommand _DeleteCommand;
    /// <summary> Удалить друга </summary>
    public ICommand DeleteCommand => _DeleteCommand ??= new Command(OnDeleteCommandExecuted);
    private void OnDeleteCommandExecuted(object p)
    {
        var friend = p as FriendViewModel;
        if (friend != null)
        {
            Friends.Remove(friend);
        }
        _navigation.PopAsync();
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
```csharp
public class FriendViewModel : INotifyPropertyChanged
{
    public Friend Friend { get; private set; }

    public FriendViewModel()
    {
        Friend = new Friend();
    }

    private FriendsListViewModel _model;
    public FriendsListViewModel ListViewModel
    {
        get => _model;
        set => Set(ref _model, value);
    }

    public string Name
    {
        get => Friend.Name;
        set
        {
            if (Equals(Friend.Name, value)) return;
            Friend.Name = value;
            OnPropertyChanged(nameof(Name));
        }
    }
    public string Email
    {
        get => Friend.Email;
        set
        {
            if (Equals(Friend.Email, value)) return;
            Friend.Email = value;
            OnPropertyChanged(nameof(Email));
        }
    }
    public string Phone
    {
        get => Friend.Phone;
        set
        {
            if (Equals(Friend.Phone, value)) return;
            Friend.Phone = value;
            OnPropertyChanged(nameof(Phone));
        }
    }

    public bool IsValid => ((!string.IsNullOrEmpty(Name)) || 
        (!string.IsNullOrEmpty(Phone)) || (!string.IsNullOrEmpty(Email)));

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
Разметка
```xml
<StackLayout Padding="5">
    <Button Text="Добавить" Command="{Binding CreateFriendCommand}" />
    <ListView x:Name="ListViewFriends" ItemsSource="{Binding Friends}" 
                SelectedItem="{Binding SelectedFriend, Mode=TwoWay}" HasUnevenRows="True">
        <ListView.ItemTemplate>
            <DataTemplate>
                <ViewCell>
                    <ViewCell.View>
                        <StackLayout>
                            <Label Text="{Binding Name}" FontSize="Medium"></Label>
                            <Label Text="{Binding Email}" FontSize="Small"></Label>
                            <Label Text="{Binding Phone}" FontSize="Small"></Label>
                        </StackLayout>
                    </ViewCell.View>
                </ViewCell>
            </DataTemplate>
        </ListView.ItemTemplate>
    </ListView>
</StackLayout>
```
```xml
<StackLayout Padding="5">
    <StackLayout x:Name="StackLayoutFriend">
        <Label Text="Имя"></Label>
        <Entry Text="{Binding Name}" FontSize="Medium"></Entry>
        <Label Text="Email"></Label>
        <Entry Text="{Binding Email}" FontSize="Medium"></Entry>
        <Label Text="Phone"></Label>
        <Entry Text="{Binding Phone}" FontSize="Medium"></Entry>
    </StackLayout>
    <StackLayout Orientation="Horizontal" HorizontalOptions="CenterAndExpand">
        <Button Text="Добавить" Command="{Binding ListViewModel.SaveCommand}" CommandParameter="{Binding}" />
        <Button Text="Удалить" Command="{Binding ListViewModel.DeleteCommand}" CommandParameter="{Binding}" />
        <Button Text="Назад" Command="{Binding Path=ListViewModel.BackCommand}" />
    </StackLayout>
</StackLayout>
```
Обработчики событий
```csharp
public FriendsListPage()
{
    InitializeComponent();
    BindingContext = new FriendsListViewModel( this.Navigation );
}
```
```csharp
public FriendViewModel ViewModel { get; private set; }
public FriendPage(FriendViewModel model)
{
    InitializeComponent();
    ViewModel = model;
    this.BindingContext = ViewModel;
}
```



## Инфраструктура

<PackageReference Include="Microsoft.Extensions.DependencyInjection" Version="6.0.0" />

App.xaml.cs
```csharp
private static IServiceProvider _Services;
private static IServiceCollection GetServices()
{
    var services = new ServiceCollection();
    InitializeServices(services);
    return services;
}
/// <summary> Сервисы приложения </summary>
public static IServiceProvider Services => _Services ??= GetServices().BuildServiceProvider();
private static void InitializeServices(IServiceCollection services)
{
    services.AddScoped<MainPageViewModel>();

    services.AddScoped<MainPage>();
}

public App()
{
    InitializeComponent();

    var mainPage = Services.GetRequiredService<MainPage>();
    mainPage.ViewModel = Services.GetRequiredService<MainPageViewModel>();

    MainPage = new NavigationPage(mainPage);
}
```

Base.Entity
```csharp
public abstract class Entity : INotifyPropertyChanged
{
    public int Id { get; set; }
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

Friend
```csharp
public class Friend : Entity
{
    public string Name { get; set; }
    public string Email { get; set; }
    public string Phone { get; set; }
}
```

Base.ViewModel
```csharp
public abstract class ViewModel : INotifyPropertyChanged
{
    public int Id { get; set; }
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

MainPageViewModel
```csharp
public class MainPageViewModel : ViewModel
{
    #region Свойства

    private string _Title = "Клиентское приложение";
    /// <summary> Название приложения </summary>
    public string Title
    {
        get => _Title;
        set => Set(ref _Title, value);
    }

    #endregion

    public MainPageViewModel()
    {

    }

    #region Команды

    #endregion
}
```

Locator
```csharp
public class ViewModelLocator
{
    public MainPageViewModel MainPageViewModel => App.Services
        .GetRequiredService<MainPageViewModel>();
}
```

App.xaml
```xml
<Application.Resources>
    <vm:ViewModelLocator x:Key="Locator" />
</Application.Resources>
```

MainPage.xaml
```xml
<ContentPage xmlns="http://xamarin.com/schemas/2014/forms"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml" 
             xmlns:vm="clr-namespace:ClientApp.ViewModels" 
             x:DataType="vm:MainPageViewModel"
             x:Class="ClientApp.Views.MainPage"
             Title="{Binding Title}"             
             BindingContext="{Binding MainPageViewModel, Source={StaticResource Locator}}">

    <StackLayout Padding="5">        

    </StackLayout>

</ContentPage>
```

MainPage.xaml.cs
```csharp
public MainPageViewModel ViewModel { get; set; }
```