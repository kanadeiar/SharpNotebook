# WPF Стили

Стиль WPF - это объект класса System.Windows.Style, который поддерживает коллекцию пар "свойство-значение". Свойство Setter обеспечивает возможность определения пар "свойство-значение". Объект стиль можно упаковывать как на уровне окна, так и на уровне приложения или в словаре ресурсов.

Наиболее важные члены класса Style:
```csharp
Triggers                Доступ к коллекции объектов триггеров, которые делают возможной фиксацию условий возникновения разнообразных событий в стиле
BasedOn                 Разрешает построить новый стиль на основе существующего
TargetType              Ограничение места применения стиля
```

Пример применения стиля - шрифта:
```csharp
<Application x:Class="WpfApp1.App"
...
             Startup="App_OnStartup" Exit="App_OnExit">
    <Application.Resources>
        <Style x:Key="BasicControlStyle">
            <Setter Property="Control.FontSize" Value="14"/>
            <Setter Property="Control.Height" Value="40"/>
            <Setter Property="Control.Cursor" Value="Hand"/>
        </Style>
    </Application.Resources>
</Application>
<StackPanel>
    <Label x:Name="labelInfo" Content="Применение стиля на надпись" Width="300" Style="{StaticResource BasicControlStyle}"/>
    <Button x:Name="buttonTest" Content="Ок, это примененеие стиля" Width="250" Style="{StaticResource BasicControlStyle}"/>
</StackPanel>
```
Добавление атрибута TargetType к открывающему дескриптору Style точно указывает, где этот стиль может быть применен. Этот стиль будет работать только с указанным типом элементов, с другим - возникнут ошибки разметки и компиляции. 

Пример примененения стиля с уточненнием типа:
```csharp
<Application.Resources>
    <Style x:Key="BasicControlStyle" TargetType="Control">
        <Setter Property="FontSize" Value="12"/>
        <Setter Property="Height" Value="40"/>
        <Setter Property="Cursor" Value="Hand"/>
    </Style>
    <Style x:Key="BigGreenButton" TargetType="Button">
        <Setter Property="FontSize" Value="20"/>
        <Setter Property="Height" Value="100"/>
        <Setter Property="Width" Value="200"/>
        <Setter Property="Background" Value="DarkGreen"/>
        <Setter Property="Foreground" Value="AliceBlue"/>
    </Style>
</Application.Resources>
<StackPanel>
    <Label x:Name="labelInfo" Content="Применение стиля на надпись" Width="500" Style="{StaticResource BasicControlStyle}"/>
    <Button x:Name="buttonTest" Content="Ок, это примененеие стиля" Width="250" Style="{StaticResource BasicControlStyle}"/>
    <Button x:Name="buttonGreen" Content="Зеленая кнопка" Style="{StaticResource BigGreenButton}"/>
</StackPanel>
```

Если в открывающем дескрипторе x:Key отсуствует, то стиль будет применен ко всем элементам указанного в TargetType стиля. Однако можно отказатся от применения стиля с помощью установки свойства Style в {x:Null}.

Пример применения стиля ко всем элементам + исключение:
```csharp
<Application.Resources>
    <Style TargetType="Label">
        <Setter Property="FontSize" Value="14"/>
        <Setter Property="Width" Value="300"/>
        <Setter Property="Height" Value="40"/>
        <Setter Property="BorderThickness" Value="3"/>
        <Setter Property="BorderBrush" Value="Red"/>
        <Setter Property="FontStyle" Value="Italic"/>
    </Style>
</Application.Resources>
<StackPanel>
    <Label x:Name="labelInfo" Content="Применение стиля на надпись"/>
    <Label x:Name="label2Info" Content="Применение стиля на надпись 2"/>
    <Label x:Name="label3Info" Content="Отказ от примененения стиля на надпись 3" Style="{x:Null}"/>
</StackPanel>
```

Новые стили можно строить на основе существующих используя свойство BasedOn. Расширяемый стиль должен иметь атрибут x:Key, для того чтоб производный стиль мог его наследовать через расширение разметки {StaticResource} или {DynamicResource}.

Пример применения нового стиля на основе существующего:
```csharp
<Application.Resources>
    <Style x:Key="BigGreenButton" TargetType="Button">
        <Setter Property="FontSize" Value="20"/>
        <Setter Property="Height" Value="100"/>
        <Setter Property="Width" Value="200"/>
        <Setter Property="Background" Value="DarkGreen"/>
        <Setter Property="Foreground" Value="AliceBlue"/>
    </Style>
    <Style x:Key="BigGreenRotate" TargetType="Button"
           BasedOn="{StaticResource BigGreenButton}">
        <Setter Property="RenderTransform">
            <Setter.Value>
                <RotateTransform Angle="20"/>
            </Setter.Value>
        </Setter>
    </Style>
</Application.Resources>
<StackPanel>
    <Button x:Name="buttonGreen" Content="Зеленая кнопка" Style="{StaticResource BigGreenRotate}"/>
</StackPanel>
```

Стили WPF могут содержать триггеры за счет упаковки объектов Trigger в коллекцию Triggers объекта Style. Триггеры свойств когда условие триггера не истинно, свойство автоматически получает стандартное значение, хотя триггеры событий не возвращаются обратно в предидущее значение по умолчанию.

Пример примененения стиля с одним триггером:
```csharp
<Application.Resources>
    <Style TargetType="TextBox">
        <Setter Property="FontSize" Value="14"/>
        <Setter Property="Width" Value="300"/>
        <Setter Property="Height" Value="40"/>
        <Setter Property="BorderThickness" Value="1"/>
        <Setter Property="BorderBrush" Value="Red"/>
        <Setter Property="FontStyle" Value="Italic"/>
        <Setter Property="Background" Value="AliceBlue"/>
        <Style.Triggers>
            <Trigger Property="IsFocused" Value="True">
                <Setter Property="Background" Value="Yellow"/>
            </Trigger>
        </Style.Triggers>
    </Style>
</Application.Resources>
<StackPanel>
    <TextBox x:Name="labelInfo" Text="123"/>
</StackPanel>
```

Стили WPF могут содержать триггеры с множеством условий.

Пример примененеия стиля с множетсвом условий триггеров:
```csharp
<Application.Resources>
    <Style TargetType="TextBox">
        <Setter Property="FontSize" Value="14"/>
        <Setter Property="Width" Value="300"/>
        <Setter Property="Height" Value="40"/>
        <Setter Property="BorderThickness" Value="1"/>
        <Setter Property="BorderBrush" Value="Red"/>
        <Setter Property="FontStyle" Value="Italic"/>
        <Setter Property="Background" Value="AliceBlue"/>
        <Style.Triggers>
            <MultiTrigger>
                <MultiTrigger.Conditions>
                    <Condition Property="IsFocused" Value="True"/>
                    <Condition Property="IsMouseOver" Value="True"/>
                </MultiTrigger.Conditions>
                <Setter Property="Background" Value="Yellow"/>
            </MultiTrigger>
        </Style.Triggers>
    </Style>
</Application.Resources>
<StackPanel>
    <TextBox x:Name="labelInfo" Text="123"/>
</StackPanel>
```

Стили WPF могут содержать триггеры с анимационной последовательностью.

Пример применения стиля с анимацией на основании условий триггера:
```csharp
<Application.Resources>
    <Style x:Key="BigGrowingButton" TargetType="Button">
        <Setter Property="Height" Value="40"/>
        <Setter Property="Width" Value="200"/>
        <Style.Triggers>
            <Trigger Property="IsMouseOver" Value="True">
                <Trigger.EnterActions>
                    <BeginStoryboard>
                        <Storyboard TargetProperty="Height">
                            <DoubleAnimation From="40" To="200" Duration="0:0:5" AutoReverse="True"/>
                        </Storyboard>
                    </BeginStoryboard>
                </Trigger.EnterActions>
            </Trigger>
        </Style.Triggers>
    </Style>
</Application.Resources>
<StackPanel>
    <TextBox x:Name="labelInfo" Text="123"/>
</StackPanel>
```

Стиль может применяться во время выполнения приложения. 

Пример примененения стиля во время выполнения приложения:
```csharp
<Application.Resources>
    <Style x:Key="ButtonStyle1" TargetType="Button">
        <Setter Property="Background" Value="Aqua"/>
    </Style>
    <Style x:Key="ButtonStyle2" TargetType="Button">
        <Setter Property="Background" Value="Red"/>
    </Style>
    <Style x:Key="ButtonStyle3" TargetType="Button">
        <Setter Property="Background" Value="LightGreen"/>
    </Style>
</Application.Resources>
<DockPanel>
    <StackPanel Orientation="Horizontal" DockPanel.Dock="Top" Margin="0,0,0,50">
        <Label Content="Установка стиля для кнопки" Height="30"/>
        <ListBox x:Name="listStyles" Height="70" Width="200" Background="LightCyan" SelectionChanged="ListStyles_OnSelectionChanged"></ListBox>
    </StackPanel>
    <Button x:Name="buttonStyle" Height="40" Width="100" Content="OK!"/>
</DockPanel>
public MainWindow()
{
    InitializeComponent();
    listStyles.Items.Add("ButtonStyle1");
    listStyles.Items.Add("ButtonStyle2");
    listStyles.Items.Add("ButtonStyle3");
}
private void ListStyles_OnSelectionChanged(object sender, SelectionChangedEventArgs e)
{
    var currentStyle = (Style) TryFindResource(listStyles.SelectedValue);
    if (currentStyle == null)
        return;
    buttonStyle.Style = currentStyle;
}
```














