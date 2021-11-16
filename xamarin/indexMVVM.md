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



