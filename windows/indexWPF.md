# Основы WPF

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
        xmlns:local="clr-namespace:WpfApp1HelloWPF"
        mc:Ignorable="d"
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



