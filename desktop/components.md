# Компоненты

Основные элементы управления для пользовательского ввода:

Button, RatioButton, ComboBox, CheckBox, Calendar, DatePicker, Expander, DataGrid, ListBox, ListView, ToggleButton, TreeView, ContextMenu, ScrollBar, Slider, TabControl, TextBlock, TextBox, RepeatButton, RichTextBox, Label

WPF предлагает полное семейство элементов управления, которые можно задействовать при построении пользовательских интерфейсов

Боковые элементы окон и элементов управления:

Menu, ToolBar, StatusBar, ToolTip, ProgressBar

Элементы для декорирования рамки объекта Window компонентами для ввода и элементами информаирования пользователя

Элементы управления мультимедиа:

Image, MediaELement, SoundPlayerAction

Элементы предоставляют поддержку воспроизведения аудио/видео и вывода изображений

Элементы управления компоновкой:

Border, Canvas, DockPanel, Grid, GridView, GridSplitter, GroupBox, Panel, TabControl, StackPanel, ViewBox, WrapPanel

WPF предлагает множество элементво управления для группировки и организации других жлементов в целях управления компоновкой

WPF предоставляет элементы для работы с интерфейсом Ink API. Данный аспект разработки WPF полезен при построении приложений для Tablet PC, т.к. он позволяет захватывать ввод от пера.

WPF предоставляет элементы для работы с документами в стиле Adobe PDF, с применением типов из пространства имен System.Windows.Documents (PresentationFramework.dll), работающие с API-интерфейсом XML Paper Specification (XPS).

WPF Предлагает несколько общих диалоговых окон, как OpenFileDialog и SaveFileDialog.

## Меню Menu.

В WPF меню представлены классом Menu, поддерживающий коллекуию объектов MenuItem, поддерживающий обработку разообразных событий.

Пример:

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

Пример:

```csharp
<ToolBar x:Name="inkToolBar" Height="60" Background="LightGreen">
    <Border VerticalAlignment="Center">
        <WrapPanel>
            <RadioButton x:Name="inkRadio" Margin="5,10" Content="InkMode" IsChecked="True" Click="inkRadio_Click"/>
            <RadioButton x:Name="eraseRadio" Margin="5,10" Content="Erase Mode" Click="inkRadio_Click"/>
            <RadioButton x:Name="selectRadio" Margin="5,10" Content="SelectMode" Click="inkRadio_Click"/>
        </WrapPanel>
    </Border>
    <Separator/>
    <ComboBox x:Name="comboColors" Width="100" SelectionChanged="comboColors_SelectionChanged">
        <ComboBoxItem Content="Красный"/>
        <ComboBoxItem Content="Зеленый"/>
        <ComboBoxItem Content="Желтый"/>
    </ComboBox>
    <Separator/>
    <Grid>
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="Auto"/>
            <ColumnDefinition Width="Auto"/>
            <ColumnDefinition Width="Auto"/>
        </Grid.ColumnDefinitions>
        <Button Grid.Column="0" x:Name="buttonSave" Height="30" Width="70" Margin="2" Content="Сохранить"/>
        <Button Grid.Column="1" x:Name="buttonLoad" Height="30" Width="70" Margin="2" Content="Загрузить"/>
        <Button Grid.Column="2" x:Name="buttonClear" Height="30" Width="70" Margin="2" Content="Очистить"/>
    </Grid>
    <Separator/>
</ToolBar>
private void RadioButton_Click(object sender, RoutedEventArgs e)
{
    MessageBox.Show("1");
}
private void comboColors_SelectionChanged(object sender, SelectionChangedEventArgs e)
{
    MessageBox.Show("2");
}
```

Пример:

```csharp
<ToolBarTray DockPanel.Dock="Top">
    <ToolBar Height="30" Band="0" ToolBarTray.IsLocked="True" Header="Сервера">
        <ComboBox MinWidth="120" MaxWidth="200" SelectedIndex="0">
            <ComboBoxItem>smtp.yandex.ru:587</ComboBoxItem>
            <ComboBoxItem>smtp.gmail.com:587</ComboBoxItem>
            <ComboBoxItem>smtp.mail.ru:25</ComboBoxItem>
        </ComboBox>
        <Button Name="ButtonAdd" Height="25" ToolTip="Добавить">
            <Image Source="Images/add.png"/>
        </Button>
        <Button Name="ButtonEdit" Height="25" ToolTip="Добавить">
            <Image Source="Images/edit.png"/>
        </Button>
        <Button Name="ButtonDelete" Height="25" ToolTip="Добавить">
            <Image Source="Images/del.png"/>
        </Button>
    </ToolBar>
    <ToolBar Header="два" Band="1" BandIndex="2">
    </ToolBar>
    <ToolBar Header="еще" Band="0">
    </ToolBar>
    <ToolBar Band="1">
    </ToolBar>
</ToolBarTray>
```

## Строка состояния StatusBar.

Элемент строка состояния StatusBar стыкуется с нижней частью окна и содержит различные элементы для отображения информации.

Пример:

```csharp
<Window ...ResizeMode="CanResizeWithGrip"...
<StatusBar DockPanel.Dock="Bottom" Background="Beige">
    <StatusBarItem DockPanel.Dock="Right" Width="20">
    </StatusBarItem>
    <StatusBarItem>
        <TextBlock Name="statusBarText" Text="Готов"/>
    </StatusBarItem>
</StatusBar>
```

## Виджет вкладок TabControl.

Виджет TabControl содержит набор вкладок TabItem, на каждой вкладке можно расставять свои элементы пользовательского интерфейса.

Пример:

```csharp
<TabControl x:Name="myTabControl" HorizontalAlignment="Stretch" VerticalAlignment="Stretch">
    <TabItem Header="Ink API">
        <Grid Background="AntiqueWhite"/>
    </TabItem>
    <TabItem Header="Data Binding">
        <Grid Background="LightGoldenrodYellow"/>
    </TabItem>
    <TabItem Header="DataGrid">
        <Grid Background="Azure"/>
    </TabItem>
</TabControl>
```

