# Основы WPF




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
Использование файла ресурсов. 

Пример файл "Dictionary1.xaml" & "MainWindow.xaml":
```csharp
<ResourceDictionary xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
...
    <LinearGradientBrush x:Key="Brush">
        <GradientStopCollection>
            <GradientStop Color="White" Offset="0"/>
            <GradientStop Color="Blue" Offset="1"/>
        </GradientStopCollection>
    </LinearGradientBrush>
</ResourceDictionary>
<Window x:Class="WpfApp1HelloWPF.MainWindow"
...
        Title="Заголовок окна" Height="600" Width="800">
    <Window.Resources>
        <ResourceDictionary Source="Dictionary1.xaml"/>
    </Window.Resources>
    <DockPanel Margin="10" Name="panelMain" Background="{StaticResource Brush}">       
    </DockPanel>
</Window>
```
# Стили

Использование стилей.

Пример использования стилей:
```csharp
<Window x:Class="WpfApp1HelloWPF.MainWindow"
...
        Title="Заголовок окна" Height="600" Width="800">
    <Window.Resources>
    </Window.Resources>
    <StackPanel Margin="10">
        <StackPanel.Resources>
            <Style TargetType="TextBlock">
                <Setter Property="Foreground" Value="Gray"/>
                <Setter Property="FontSize" Value="26"/>
            </Style>
        </StackPanel.Resources>
        <TextBlock>Заголовок</TextBlock>
        <TextBlock Foreground="Blue">Еще заголовок</TextBlock>
        <TextBlock Text="Style Test">
            <TextBlock.Style>
                <Style>
                    <Setter Property="TextBlock.FontSize" Value="40"/>
                </Style>
            </TextBlock.Style>
        </TextBlock>
    </StackPanel>
</Window>
```
Стили для окна:
```csharp
<Window x:Class="WpfApp1HelloWPF.MainWindow"
...
        Title="Заголовок окна" Height="600" Width="800">
    <Window.Resources>
        <Style TargetType="TextBlock">
            <Setter Property="Foreground" Value="Gray"/>
            <Setter Property="FontSize" Value="20"/>
        </Style>
    </Window.Resources>
    <StackPanel Margin="10">
        <TextBlock>Заголовок окна</TextBlock>
    </StackPanel>
</Window>
```

Стили для всего приложения - файлы "App.xaml" & "MainWindow.xaml":
```csharp
<Application x:Class="WpfApp1HelloWPF.App"
...
             StartupUri="MainWindow.xaml">
    <Application.Resources>
        <Style TargetType="TextBlock">
            <Setter Property="Foreground" Value="Gray"/>
            <Setter Property="FontSize" Value="20"/>
        </Style>
    </Application.Resources>
</Application>
<Window x:Class="WpfApp1HelloWPF.MainWindow"
...
        Title="Заголовок окна" Height="600" Width="800">
    <StackPanel Margin="10">
        <TextBlock>Заголовок окна</TextBlock>
    </StackPanel>
</Window>
```

Пример явного применения стилей - файлы "App.xaml" & "MainWindow.xaml":
```csharp
<Application x:Class="WpfApp1HelloWPF.App"
...
             StartupUri="MainWindow.xaml">
    <Application.Resources>
        <Style TargetType="TextBlock" x:Key="HeaderStyle">
            <Setter Property="Foreground" Value="Gray"/>
            <Setter Property="FontSize" Value="20"/>
        </Style>
    </Application.Resources>
</Application>
<Window x:Class="WpfApp1HelloWPF.MainWindow"
...
        Title="Заголовок окна" Height="600" Width="800">
    <Window.Resources>
    </Window.Resources>
    <StackPanel Margin="10">
        <TextBlock Style="{StaticResource HeaderStyle}">Заголовок окна</TextBlock>
    </StackPanel>
</Window>
```

## Обработка исключений

Пример перехвата глобальных исключений:
```csharp
//App.xaml
<Application x:Class="WpfApp1HelloWPF.App"
...
             DispatcherUnhandledException="Application_DispatcherUnhandledException"
             StartupUri="MainWindow.xaml">
    <Application.Resources>

    </Application.Resources>
</Application>
//App.xaml.cs
private void Application_DispatcherUnhandledException(object sender, System.Windows.Threading.DispatcherUnhandledExceptionEventArgs e)
{
    MessageBox.Show($"Исключение: {e.Exception.Message}", "Исключение", MessageBoxButton.OK,MessageBoxImage.Warning);
}
//MainWindow.xaml
<Window x:Class="WpfApp1HelloWPF.MainWindow"
...
        Title="Заголовок окна" Height="400" Width="600" WindowStartupLocation="CenterScreen">
    <Grid>
        <Button x:Name="buttonMy" Content="Моя кнопка" HorizontalAlignment="Center" VerticalAlignment="Center" Width="150" Click="buttonMy_Click"/>
    </Grid>
</Window>
private void buttonMy_Click(object sender, RoutedEventArgs e)
{
    string s = null;
    int length = s.Length;
}
```

