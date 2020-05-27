# WPF Позиционирование (панели, прокрутка, меню, панель элементов, строка состояния)

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

Панель Canvas.

Панель Canvas делает возможным абсолютное позиционирование содержимого пользовательского содержимого. Для каждого элемента управления необходимо указать левый верхний угол с симпользованием Canvas.Top & Cancas.Left. Нижний правый угол каждого элемента управления можно задать неявно, устанавливая свойства Canvas.Height & Canvas.Width, либо свойствами Canvas.Roght & Canvas.Bottom. Элементы внутри Canvas не изменяют свои размеры динамически ии при использовании стилей или шаблонов. Панель Canvas не пытается сохранять элементы видимыми, когда конечный пользователь уменьшает размер окна.

Пример позиционирования элементов в панели Canvas:
```csharp
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
```

Панель WrapPanel.

Панель WrapPanel позволяет помещать содержимое, протекающее сквозь панель, когда размеры его изменяются. Для каждого элемента определяются значения свойств Height & Width, чтоб управлять их общим поведением в контейнере. Порядок объявления элементов играет важную роль. При изменении ширины окна содержимое выглядит не особо привлекательно, так как оно протекает слева направо внутри окна. Изменением свойства Orientation="Vertical" можно заставить содержимое протекать сверху вниз.

Пример позиционирования в панели WrapPanel:
```csharp
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
```

Помимо этого панель может быть объявлена с указанием значений ItemWidth & ItemHeight, которые управляют стандартным размером каждого элемента, если подэлемент не предоставляет собственные значение своей ширины и высоты.

Пример позиционирования элементов в панели Canvas:
```csharp
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
```

Панель StackPanel.

Панель StackPanel организует содержимое внутри одиночной строки, которая может быть ориентирована горизонтально или вертикально в зависимости от значения Orientation. StackPanel не переносит содержимое при изменении размера окна пользователем. Элементы просто растягиваются, приспосабливаясь к размерам панели StackPanel.

Пример использования панели StackPanel:
```csharp
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
```

Панель Grid.

Панель Grid является самой гибкой. Она может состоять из набора ячеек, каждая из которых имеет свое содержимое. По умолчанию элемент Grid состоит из единственной ячейки, заполняющей всю поверхность окна.

Порядок определения этой панели:

1. Определение каждой колонки. Используется элемент Grid.ColumnDefinitions с коллекцией ColumnDefinition.

2. Определение каждой строки. Используется элемент Grid.RowDefinitions с коллекцией RowDefinition.

3. Назначение содержимого каждой ячеке сетки с применением синтаксиса Присоединяемых свойств.

Пример определения панели Grid:
```csharp
<Grid ShowGridLines="True" Background="LightSteelBlue">
    <Grid.ColumnDefinitions><ColumnDefinition/><ColumnDefinition/></Grid.ColumnDefinitions>
    <Grid.RowDefinitions><RowDefinition/><RowDefinition/></Grid.RowDefinitions>
    <Label x:Name="labelInstructions" Grid.Column="0" Grid.Row="0" FontSize="14" Content="Введите информацию:"/>
    <Label x:Name="labelMake" Grid.Column="1" Grid.Row="0" Content="Марка:"/>
    <TextBox x:Name="boxMake" Grid.Column="1" Grid.Row="0" Height="30"/>
    <Label x:Name="labelColor" Grid.Column="0" Grid.Row="1" Content="Цвет:"/>
    <TextBox x:Name="boxColor" Grid.Column="0" Grid.Row="1" Height="30"/>
    <Label x:Name="labelFormat" Grid.Column="1" Grid.Row="1" Content="Формат:"/>
    <TextBox x:Name="boxFormat" Grid.Column="1" Grid.Row="1" Height="30"/>
    <Button x:Name="buttonOk" Grid.Column="0" Grid.Row="0" Height="30" Width="100" Content="OK"/>
</Grid>
```

Размеры строк и столбцов можно указывать тремя способами:

- абсолютные размеры (100)

- автоматические размеры

- относительные размеры (3*)

Пример (первая колонка - 25%, а вторая - 75%):
```csharp
<Grid.ColumnDefinitions>
    <ColumnDefinition Width="1*"/>
    <ColumnDefinition Width="3*"/>
</Grid.ColumnDefinitions>
```

Панели Grid поддерживают разделители GridSplitter, позволяющие пользователю приложения изменять размеры колонок и строк сетки. При этом содержимое каждой ячейки реорганизует себя на основе находящихся в нем элементов. Каждому разделителю необходимо будет установить свойство Width или Height в зависимости от вертикального или горизонтального разделителя. Колонка или строка, поддерживающие разделитель, должны быть установлены своствами Width или Height в Auto.

Пример разметки с разделителем:
```csharp
<Grid ShowGridLines="True" Background="LightSteelBlue">
    <Grid.ColumnDefinitions>
        <ColumnDefinition Width="1*"/>
        <ColumnDefinition Width="3*"/>
    </Grid.ColumnDefinitions>
    <Grid.RowDefinitions>
        <RowDefinition/>
        <RowDefinition/>
    </Grid.RowDefinitions>
    <GridSplitter Grid.Column="0" Grid.RowSpan="2" Width="5"/>
    <Label Content="Left!"/>
    <Label Content="Right!" Grid.Column="1" Grid.Row="1"/>
</Grid>
```

Панель DockPanel.

Панель DockPanel является контейнер, содержащий любое количество дополнительных панелей для группировки связанного содержимого. Эта панель использует синтаксис присоединяемых свойств для управления местом, куда будет пристыковаватся каждый элемент внутри панели.

Пример разметки:
```csharp
<DockPanel LastChildFill="True" Background="LightSteelBlue"> <!--LastChildFill - последний элемент заполняет всю оставшуюся область-->
    <Label DockPanel.Dock="Top" Content="Заголовок"/>
    <Label DockPanel.Dock="Left" Content="Марка"/>
    <Label DockPanel.Dock="Right" Content="Цвет"/>
    <Label DockPanel.Dock="Bottom" Content="Название"/>
    <Button x:Name="buttonOk" Content="OK"/>
</DockPanel>
```

## Прокрутка ScrollViewer.

Класс прокрутки ScrollViewer позволяет обеспечить автоматическое поведение прокрутки данных внутри объектов панелей.

Пример использования прокрутки:
```csharp
<ScrollViewer Padding="10">
    <StackPanel>
        <Label Content="Текст" Background="Green" Height="80"/>
        <Button Content="OK" Background="Yellow" Height="80"/>
        <Label Content="Еще текст" Height="80"/>
        <Button Content="Cancel" Background="Aqua" Height="80"/>
        <Button Content="Esc" Height="80"/>
    </StackPanel>
</ScrollViewer>
```

## Меню Menu.

В WPF меню представлены классом Menu, поддерживающий коллекуию объектов MenuItem, поддерживающий обработку разообразных событий.

Пример определения меню Menu:
```csharp
<DockPanel Background="LightSteelBlue">
    <Menu DockPanel.Dock="Top" HorizontalAlignment="Left" Background="LightGray" BorderBrush="Black">
        <MenuItem Header="Файл">
            <MenuItem Header="Создать"/>
            <Separator/>
            <MenuItem Header="Выход" Click="MenuItemExit_OnClick"/>
        </MenuItem>
        <MenuItem Header="Инструменты">
            <MenuItem Header="Подсказка"/>
        </MenuItem>
    </Menu>
</DockPanel>
private void MenuItemExit_OnClick(object sender, RoutedEventArgs e)
{
    this.Close();
}
```

## Панель инструментов ToolBar.

Панель инструментов ToolBar предлагают альтернативный способ активизауии пунктов меню. С помощью него можно дублировать команды из главного меню приложения. На его поверхность можно помещать любые типы - раскрывающиеся списки, изображения и графику.

Пример:
```csharp
<ToolBar DockPanel.Dock="Top">
    <Button Content="Выход" MouseEnter="UIElement_OnMouseEnter" MouseLeave="UIElement_OnMouseLeave" Click="MenuItemExit_OnClick"/>
    <Separator/>
    <Button Content="Проверка" MouseEnter="UIElement_OnMouseEnter1" MouseLeave="UIElement_OnMouseLeave" Click="MenuItem_Click"/>
</ToolBar>
```


## Строка состояния StatusBar.

Элемент строка состояния StatusBar стыкуется с нижней частью окна и содержит различные элементы для отображения информации.

Пример разметки строки состояния SatausBar:
```csharp
<StatusBar DockPanel.Dock="Bottom" Background="Beige">
    <StatusBarItem>
        <TextBlock Name="statusBarText" Text="Готов"/>
    </StatusBarItem>
</StatusBar>
```






