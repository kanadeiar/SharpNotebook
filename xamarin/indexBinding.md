# Привязки Xamarin.Forms

## Базовое приложение

Как и в WPF приложениях, здесь классы, доступные для привязок должны реализовывать INotifyPropertyChanged. 

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
