# Основы WPF

Стартовое окно по умолчанию - файл "App.xaml":
```csharp
<Application x:Class="WpfApp1HelloWPF.App"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:local="clr-namespace:WpfApp1HelloWPF"
             StartupUri="MainWindow.xaml">
    <Application.Resources> </Application.Resources>
</Application>
```
Процедура отображения стартового окна - файл "App.xaml" & "App.xaml.cs":
```csharp
<Application x:Class="WpfApp1HelloWPF.App"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
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



