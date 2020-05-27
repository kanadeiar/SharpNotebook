# WPF Примеры

## Простые примеры кода:

Стартовое окно по умолчанию - файл "App.xaml":
```csharp
<Application x:Class="WpfApp1HelloWPF.App"
...
             StartupUri="MainWindow.xaml">
    <Application.Resources> </Application.Resources>
</Application>
```
Процедура отображения стартового окна - файл "App.xaml" & "App.xaml.cs":
```csharp
<Application x:Class="WpfApp1HelloWPF.App"
....
             xmlns:local="clr-namespace:WpfApp1HelloWPF"
             Startup="Application_Startup">
    <Application.Resources> </Application.Resources>
</Application>
//процедура во втором файле:
public partial class App : Application
{
    private void Application_Startup(object sender, StartupEventArgs e)
    {
        MainWindow wnd = new MainWindow();
        wnd.Title = "Привет WPF!";
        wnd.Show();
    }
}
```
Получение списка параметров при запуске приложения:
```csharp
private void Application_Startup(object sender, StartupEventArgs e)
{
    MainWindow wnd = new MainWindow();
    foreach (var el in e.Args)
        MessageBox.Show($"Параметр: {el}");
    wnd.Show();
}
```
Событие - клик по кнопке. Пример:
```csharp
<Window x:Class="WpfApp1HelloWPF.MainWindow"
....
        Title="MainWindow" Height="600" Width="800">
    <Grid x:Name="MainGrid" MouseUp="MainGrid_MouseUp" Background="LightBlue">
        <Button Click="Button_Click" FontWeight="Bold" FontSize="20" Width="300" Height="50">
            <WrapPanel Name="pnlMain">
                <TextBlock Foreground="Blue">Разно</TextBlock>
                <TextBlock Foreground="Red">цветная</TextBlock>
                <TextBlock>кнопка</TextBlock>
            </WrapPanel>
        </Button>
    </Grid>
</Window>
//обработчики нажатий:
private void Button_Click(object sender, RoutedEventArgs e)
{
    MessageBox.Show("dd ");
}

private void MainGrid_MouseUp(object sender, MouseButtonEventArgs e)
{
    MessageBox.Show($"Gt: {e.GetPosition(this).ToString()}");
}
```

## Текстовый редактор
```csharp
<DockPanel Background="LightSteelBlue">
    <Menu DockPanel.Dock="Top" HorizontalAlignment="Left" Background="LightGray" BorderBrush="Black">
        <MenuItem Header="Файл">
            <MenuItem Header="Создать"/>
            <Separator/>
            <MenuItem Header="Выход" Click="MenuItemExit_OnClick" MouseEnter="UIElement_OnMouseEnter" MouseLeave="UIElement_OnMouseLeave"/>
        </MenuItem>
        <MenuItem Header="Инструменты">
            <MenuItem Header="Подсказка" MouseEnter="UIElement_OnMouseEnter1" MouseLeave="UIElement_OnMouseLeave" Click="MenuItem_Click"/>
        </MenuItem>
    </Menu>
    <StatusBar DockPanel.Dock="Bottom" Background="Beige">
        <StatusBarItem>
            <TextBlock Name="statusBarText" Text="Готов"/>
        </StatusBarItem>
    </StatusBar>
    <Grid DockPanel.Dock="Left" Background="AliceBlue">
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="Auto"/>
            <ColumnDefinition/>
        </Grid.ColumnDefinitions>
        <GridSplitter Grid.Column="0" Width="5" Background="Gray"/>
        <StackPanel Grid.Column="0" VerticalAlignment="Stretch">
            <Label Name="labelSpellingInts" FontSize="14" Margin="5">
                Подсказки
            </Label>
            <Expander x:Name="expanderSpelling" Header="Нажми здесь" Margin="5">
                <Label Name="labelSpelling" FontSize="12" Margin="5"/>
            </Expander>
        </StackPanel>
        <TextBox Grid.Column="1" SpellCheck.IsEnabled="True" AcceptsReturn="True" Name="textData" FontSize="14" BorderBrush="Blue" VerticalScrollBarVisibility="Auto" HorizontalScrollBarVisibility="Auto"></TextBox>
    </Grid>
</DockPanel>
public MainWindow()
{
    InitializeComponent();
}
private void MenuItemExit_OnClick(object sender, RoutedEventArgs e)
{
    this.Close();
}
private void UIElement_OnMouseEnter(object sender, MouseEventArgs e)
{
    statusBarText.Text = "Выход из приложения";
}
private void UIElement_OnMouseEnter1(object sender, MouseEventArgs e)
{
    statusBarText.Text = "Показ подсказки";
}
private void UIElement_OnMouseLeave(object sender, MouseEventArgs e)
{
    statusBarText.Text = "Готов";
}
private void MenuItem_Click(object sender, RoutedEventArgs e)
{
    string spellingHints = string.Empty;
    SpellingError error = textData.GetSpellingError(textData.CaretIndex);
    if (error != null)
    {
        foreach (var s in error.Suggestions)
        {
            spellingHints += $"{s}\n";
        }
        labelSpellingInts.Content = spellingHints;
        expanderSpelling.IsExpanded = true;
    }
}
```







