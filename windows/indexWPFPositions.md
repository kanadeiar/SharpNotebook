# WPF Позиционирование (панели)

Некоторые элементы управления типа панелей WPF

```csharp
Панели              Описание
Canvas              Классический режим размещения содержимого. Элементы остаются в точности там, куда были помещены
DockPanel           Привязывает содержимое к указанной стороне панели - Top, Bottom, Left, Right
Grid                Содержимое внутри серии ячеек, поддерживаемых внутри табличной сетки
StackPanel          Укладывает содержимое вертикально или горизонтально, как регламинтируется Orientation
WrapPanel           Позиционирует содержимое слева направо, перенося его на следующую строку по достижении границы панели, дальнейшее просиходит последовательно сверху вниз или слева направа в зависимости от Orientation
```

WPF приложение неизменно содержит большое количество элементов пользовательского интерфейса, которые должны быть хорошо организованы внутри окон. Чтобы сохранить элемент управления WPF внутри позиций другого окна, можно задействовать множетсо типов панелей (диспетчеры компоновки).

По умолчанию в каждом новом окре главный элемент будет применять диспетчер компоновки типа Grid. Объявленный элемент управления внутри окна позиционируется по центру контейнера. При этом помещение внутрь области главного окна более одного объекта запрещено, так как свойству Content может быть присвоено только одно значение:
```csharp
<Window x:Class="WpfApp1.MainWindow"
...
        Title="MainWindow" Height="400" Width="600">
    <Button Name="ButtonOK" Height="100" Width="200" Content="OK"/>
</Window>
```
Элементы управления типа панелей разрешено помещсть внутрь других панелей, чтоб обеспечить гибкость и управление приложения.

Пример:
```csharp
<Window x:Class="WpfApp1.MainWindow"
...
        Title="MainWindow" Height="400" Width="600">
    <Grid>
        <StackPanel>
            <Label Content="раз"></Label>
            <Label Content="два"></Label>
        </StackPanel>
    </Grid>
</Window>
```

## Панели

Панель Canvas делает возможным абсолютное позиционирование содержимого пользовательского содержимого. Для каждого элемента управления необходимо указать левый верхний угол с симпользованием Canvas.Top & Cancas.Left. Нижний правый угол каждого элемента управления можно задать неявно, устанавливая свойства Canvas.Height & Canvas.Width, либо свойствами Canvas.Roght & Canvas.Bottom. Элементы внутри Canvas не изменяют свои размеры динамически ии при использовании стилей или шаблонов. Панель Canvas не пытается сохранять элементы видимыми, когда конечный пользователь уменьшает размер окна.

Пример позиционирования элементов в панели Canvas:
```csharp
<Window x:Class="WpfApp1.MainWindow"
...
        Title="MainWindow" Height="400" Width="600">
    <Canvas Background="LightSteelBlue">
        <Button x:Name="buttonOk" Canvas.Left="212" Canvas.Top="203" Width="80" Content="OK"/>
        <Label x:Name="labelInstructions" Canvas.Left="17" Canvas.Top="14" Width="328" Height="28" FontSize="14" Content="Введите информацию:"/>
        <Label x:Name="labelMake" Canvas.Left="17" Canvas.Top="60" Content="Марка:"/>
        <Label x:Name="labelColor" Canvas.Left="17" Canvas.Top="110" Content="Цвет:"/>
        <Label x:Name="labelFormat" Canvas.Left="17" Canvas.Top="155" Content="Формат:"/>
        <TextBox x:Name="boxMake" Canvas.Left="100" Canvas.Top="60" Width="200" Height="26"/>
        <TextBox x:Name="boxColor" Canvas.Left="100" Canvas.Top="110" Width="200" Height="26"/>
        <TextBox x:Name="boxFormat" Canvas.Left="100" Canvas.Top="155" Width="200" Height="26"/>
    </Canvas>
</Window>
```

Панель WrapPanel позволяет помещать содержимое, протекающее сквозь панель, когда размеры его изменяются. Для каждого элемента определяются значения свойств Height & Width, чтоб управлять их общим поведением в контейнере. Порядок объявления элементов играет важную роль. При изменении ширины окна содержимое выглядит не особо привлекательно, так как оно протекает слева направо внутри окна. Изменением свойства Orientation="Vertical" можно заставить содержимое протекать сверху вниз.

Пример позиционирования в панели WrapPanel:
```csharp
<Window x:Class="WpfApp1.MainWindow"
...
        Title="MainWindow" Height="400" Width="500">
    <WrapPanel Background="LightSteelBlue">
        <Label x:Name="labelInstructions" Canvas.Left="17" Canvas.Top="14" Width="328" Height="28" FontSize="14" Content="Введите информацию:"/>
        <Label x:Name="labelMake" Content="Марка:"/>
        <TextBox x:Name="boxMake" Width="200" Height="26"/>
        <Label x:Name="labelColor" Content="Цвет:"/>
        <TextBox x:Name="boxColor" Width="200" Height="26"/>
        <Label x:Name="labelFormat" Content="Формат:"/>
        <TextBox x:Name="boxFormat" Width="200" Height="26"/>
        <Button x:Name="buttonOk" Width="80" Content="OK"/>
    </WrapPanel>
</Window>
```

Помимо этого панель может быть объявлена с указанием значений ItemWidth & ItemHeight, которые управляют стандартным размером каждого элемента, если подэлемент не предоставляет собственные значение своей ширины и высоты.

Пример позиционирования элементов в панели Canvas:
```csharp
<Window x:Class="WpfApp1.MainWindow"
...
        Title="MainWindow" Height="400" Width="500">
    <WrapPanel Background="LightSteelBlue" ItemWidth="200" ItemHeight="30">
        <Label x:Name="labelInstructions" FontSize="14" Content="Введите информацию:"/>
        <Label x:Name="labelMake" Content="Марка:"/>
        <TextBox x:Name="boxMake"/>
        <Label x:Name="labelColor" Content="Цвет:"/>
        <TextBox x:Name="boxColor"/>
        <Label x:Name="labelFormat" Content="Формат:"/>
        <TextBox x:Name="boxFormat"/>
        <Button x:Name="buttonOk" Content="OK"/>
    </WrapPanel>
</Window>
```

Панель StackPanel организует содержимое внутри одиночной строки, которая может быть ориентирована горизонтально или вертикально в зависимости от значения Orientation. StackPanel не переносит содержимое при изменении размера окна пользователем. Элементы просто растягиваются, приспосабливаясь к размерам панели StackPanel.

Пример использования панели StackPanel:
```csharp
<Window x:Class="WpfApp1.MainWindow"
...
        Title="MainWindow" Height="400" Width="500">
    <StackPanel Background="LightSteelBlue">
        <Label x:Name="labelInstructions" FontSize="14" Content="Введите информацию:"/>
        <Label x:Name="labelMake" Content="Марка:"/>
        <TextBox x:Name="boxMake"/>
        <Label x:Name="labelColor" Content="Цвет:"/>
        <TextBox x:Name="boxColor"/>
        <Label x:Name="labelFormat" Content="Формат:"/>
        <TextBox x:Name="boxFormat"/>
        <Button x:Name="buttonOk" Content="OK"/>
    </StackPanel>
</Window>
```





