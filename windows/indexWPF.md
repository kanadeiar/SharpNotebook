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

## Ресурсы

Пример использования статические ресурсы:
```csharp
<Window x:Class="WpfApp1HelloWPF.MainWindow"
...
        Title="MainWindow" Height="600" Width="800">
    <Window.Resources>
        <sys:String x:Key="strHello" >Hello WPF!</sys:String>
    </Window.Resources>
    <StackPanel Margin="10">
        <TextBlock Text="{StaticResource strHello}" FontSize="56"/>
    </StackPanel>
</Window>
```
Динимаческие ресурсы могут изменяться в процессе выполения программы. Пример:
```csharp
<Window x:Class="WpfApp1HelloWPF.MainWindow"
...
        Title="MainWindow" Height="600" Width="800"
        Background="{DynamicResource WindowBackgroundBrush}">
    <Window.Resources>
        <sys:String x:Key="ComboBoxTitle" >Список:</sys:String>
        <x:Array x:Key="ComboBoxItems" Type="sys:String">
            <sys:String>Элемент 1</sys:String>
            <sys:String>Элемент 2</sys:String>
            <sys:String>Элемент 3</sys:String>
        </x:Array>
        <LinearGradientBrush x:Key="WindowBackgroundBrush">
            <GradientStop Offset="0" Color="Silver"/>
            <GradientStop Offset="1" Color="LightGreen"/>
        </LinearGradientBrush>
    </Window.Resources>
    <StackPanel Margin="10">
        <Label Content="{DynamicResource ComboBoxTitle}" FontSize="20"/>
        <ComboBox ItemsSource="{DynamicResource ComboBoxItems}" FontSize="20"/>
    </StackPanel>
</Window>
```
Ресурсы могут устанавливатся на разных уровнях приложения - локально, на уровне окна или на уровне приложения.

Локально:
```csharp
<Window x:Class="WpfApp1HelloWPF.MainWindow"
...
        Title="MainWindow" Height="600" Width="800">
    <StackPanel Margin="10">
        <StackPanel.Resources>
            <sys:String x:Key="ComboBoxTitle" >Локальный рерурс</sys:String>
        </StackPanel.Resources>
        <Label Content="{DynamicResource ComboBoxTitle}" FontSize="20"/>
    </StackPanel>
</Window>
```
На уровне окна:
```csharp
<Window x:Class="WpfApp1HelloWPF.MainWindow"
...
        Title="MainWindow" Height="600" Width="800">
    <Window.Resources>
        <x:Array x:Key="ComboBoxItems" Type="sys:String">
            <sys:String>Элемент 1</sys:String>
            <sys:String>Элемент 2</sys:String>
            <sys:String>Элемент 3</sys:String>
        </x:Array>
    </Window.Resources>
    <StackPanel Margin="10">
        <ComboBox ItemsSource="{DynamicResource ComboBoxItems}" FontSize="20"/>
    </StackPanel>
</Window>
```
На уровне приложения - файл "App.xaml" и "MainWindow.xaml":
```csharp
<Application x:Class="WpfApp1HelloWPF.App"
...
    <Application.Resources>
        <sys:String x:Key="ComboBoxTitle" >Список:</sys:String>
    </Application.Resources>
</Application>
//второй файл:
<Window x:Class="WpfApp1HelloWPF.MainWindow"
...
        Title="MainWindow" Height="600" Width="800">
    <StackPanel Margin="10">
        <Label Content="{DynamicResource ComboBoxTitle}" FontSize="20"/>
    </StackPanel>
</Window>
```
Изменение ресурсов из кода. Пример:
```csharp
<Application x:Class="WpfApp1HelloWPF.App"
...
             StartupUri="MainWindow.xaml">
    <Application.Resources>
        <sys:String x:Key="appStr">Мое приложение</sys:String>
    </Application.Resources>
</Application>
<Window x:Class="WpfApp1HelloWPF.MainWindow"
...
    Title="{DynamicResource winStr}" Height="600" Width="800">
    <Window.Resources>
        <sys:String x:Key="winStr">Окно приложения</sys:String>
    </Window.Resources>
    <DockPanel Margin="10" Name="panelMain">
        <DockPanel.Resources>
            <sys:String x:Key="pnlStr">Панель</sys:String>
        </DockPanel.Resources>
        <WrapPanel DockPanel.Dock="Top" HorizontalAlignment="Center">
            <Label x:Name="labelTop" Content="{DynamicResource pnlStr}"></Label>
            <Button x:Name="buttonClickMe" Click="buttonClickMe_Click">Нажми меня</Button>
        </WrapPanel>
        <ListBox x:Name="lbResult"/>
    </DockPanel>
</Window>
private void buttonClickMe_Click(object sender, RoutedEventArgs e)
{
    lbResult.Items.Add(panelMain.FindResource("appStr").ToString());
    lbResult.Items.Add(panelMain.FindResource("winStr").ToString());
    lbResult.Items.Add(panelMain.FindResource("pnlStr").ToString());
}
```




