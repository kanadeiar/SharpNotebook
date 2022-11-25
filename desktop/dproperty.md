# Свойства зависимости

Инфраструктура WPF поддерживает уникальную программную концепцию - свойство зависимости, выглядящие как обычные свойства, но за кулисами они реализуют специальные свойства для служб WPF, требующими взаимодействия через свойства зависимости. Также предназначены для инкапсуляции полей данных класса, могут быть настроены на только для чтения, только для записи, либо оба. Свойства Height,Width,Content на самом деле являются свойствами зависимости. 

Класс, который определяет свойство зависимости, должен иметь в своей цепочке наследования класс DependencyObject. Свойство зависимости представляется как открытое, статическое, только для чтения поле DependencyProperity. В классе будет определено дружественное к XAML свойство CLR, которое вызывает методы, предоставляемые классом DependencyObject, для получения и установки значения.

Никода не нужно помещать логику проверки достоверности в блок set и оболочка CLR свойства зависимости не должна делать ничего кроме вызовов GetValue() или SetValue().

- Свойства зависимости могут наследовать свои значения от определения XAML родительского элемента.

- Свойства зависимости поддерживают возможность получать значения, установленные элементами внутри их области определения XAML.

- Свойства зависимости дают WPF способность вычислять значения на основе множества внешних значений.

- Свойства зависимости дают поддержку инфраструктуры для триггеров WPF.

- Взаимодействовать с существующим свойством зависимости можно в манере, идентичной работе с обычным свойством CLR.

## Использование

Пример определения и регистрации свойства зависимости в пользовательском элементе управления:

```csharp
<UserControl x:Class="Wpf.Controls.NumberControl"
...
    xmlns:local="clr-namespace:Wpf.Controls"
    mc:Ignorable="d"
    d:DesignHeight="300" d:DesignWidth="300">
    <StackPanel>
        <Label x:Name="LabelNumber" Height="50" Width="200" Background="LightBlue" 
            Content="{Binding CurrentNumber, RelativeSource={RelativeSource AncestorType=local:NumberControl}}"/>
    </StackPanel>
</UserControl>
public partial class NumberControl : UserControl
{
    public static readonly DependencyProperty CurrentNumberProperty =
        DependencyProperty.Register("CurrentNumber",
        typeof(int),
        typeof(NumberControl),
        new UIPropertyMetadata(default(int)));
    [Description("Текущий номер")]
    public int CurrentNumber
    {
        get => (int)GetValue(CurrentNumberProperty);
        set => SetValue(CurrentNumberProperty, value);
    }
...
}
```

Пример использования этого компонента в окне приложения:

```csharp
xmlns:myc="clr-namespace:Wpf.Controls"
...
<StackPanel>
    <myc:NumberControl x:Name="NumControl" CurrentNumber="100"/>
    <TextBox Text="{Binding ElementName=NumControl, Path=CurrentNumber, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}"></TextBox>
</StackPanel>
```

## Дополнительный функционал

Можно добавить в свойство зависимости проверку значения, приведение значения и уведомление о изменении.

Пример с приведением значения, проверкой и уведомлением:

```csharp
<UserControl x:Class="Wpf.Controls.NumberControl"
...
    mc:Ignorable="d"
    d:DesignHeight="300" d:DesignWidth="300">
    <StackPanel>   
        <Label x:Name="LabelNumber" Height="50" Width="200" Background="LightBlue" />
    </StackPanel>
</UserControl>
public partial class NumberControl : UserControl
{
    public static readonly DependencyProperty CurrentNumberProperty =
        DependencyProperty.Register("CurrentNumber",
        typeof(int),
        typeof(NumberControl),
        new UIPropertyMetadata(0, new PropertyChangedCallback(CurrentNumberChanged), new CoerceValueCallback(CoerceCurrentNumber)), new ValidateValueCallback(ValidateCurrentNumber));
    private static void CurrentNumberChanged(DependencyObject depObj,
        DependencyPropertyChangedEventArgs args)
    {
        var c = (NumberControl)depObj;
        var theLabel = c.LabelNumber;
        theLabel.Content = args.NewValue.ToString();
    }
    private static object CoerceCurrentNumber(DependencyObject d, object value)
    {
        var baseVal = (NumberControl)d;
        if ((int)value < default(int))
        {
            return default(int);
        }
        if ((int)value > 1000)
        {
            return 1000;
        }
        return value;
    }
    public static bool ValidateCurrentNumber(object value) =>
        Convert.ToInt32(value) >= 0 && Convert.ToInt32(value) <= 9000;
    [Description("Текущий номер")]
    public int CurrentNumber
    {
        get => (int)GetValue(CurrentNumberProperty);
        set => SetValue(CurrentNumberProperty, value);
    }
...
}
```

## Дополнительное

А если необходимо установить привязку данных в коде, нужно вызывать метод SetBinding():

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

