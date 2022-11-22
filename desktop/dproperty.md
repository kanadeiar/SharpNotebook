# Свойства зависимости

Инфраструктура WPF поддерживает уникальную программную концепцию - свойство зависимости, выглядящие как обычные свойства, но за кулисами они реализуют специальные свойства для служб WPF, требующими взаимодействия через свойства зависимости. Также предназначены для инкапсуляции полей данных класса, могут быть настроены на только для чтения, только для записи, либо оба. Свойства Height,Width,Content на самом деле являются свойствами зависимости. 

Класс, который определяет свойство зависимости, должен иметь в своей цепочке наследования класс DependencyObject. Свойство зависимости представляется как открытое, статическое, только для чтения поле DependencyProperity. В классе будет определено дружественное к XAML свойство CLR, которое вызывает методы, предоставляемые классом DependencyObject, для получения и установки значения.

Никода не нужно помещать логику проверки достоверности в блок set и оболочка CLR свойства зависимости не должна делать ничего кроме вызовов GetValue() или SetValue().

- Свойства зависимости могут наследовать свои значения от определения XAML родительского элемента.

- Свойства зависимости поддерживают возможность получать значения, установленные элементами внутри их области определения XAML.

- Свойства зависимости дают WPF способность вычислять значения на основе множества внешних значений.

- Свойства зависимости дают поддержку инфраструктуры для триггеров WPF.

- Взаимодействовать с существующим свойством зависимости можно в манере, идентичной работе с обычным свойством CLR.

Пример определения и регистрации свойства зависимости в элементе управления:

```csharp
public class TestElement : UIElement
{
    public static readonly DependencyProperty MergeProperty;
    public Thickness Sample
    {
        get => (Thickness)GetValue(MergeProperty);
        set => SetValue(MergeProperty, value);
    }
    static TestElement()
    {
        var metadata = new FrameworkPropertyMetadata(new Thickness());
        MergeProperty = DependencyProperty.Register("Merge", typeof(Thickness), typeof(FrameworkElement), metadata);
    }
}
```

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

При регистрации свойства зависимости нужно использовать делегат ValidateValueCallback для указания на метод, который выполняет проверку достоверности данных.

