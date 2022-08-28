# Ресурсы

Инфраструктура WPF поддерживает два типа ресурсов - двоичные ресурсы (встроенные файлы изображений или звуковых клипов, значки, используемые приложением и т.д.) и объектные ресурсы (или логические ресурсы)(именованные объекты .NET, графические данные произвольного рода).

## Двоичные ресурсы

Элемент управления Image WPF может использоватся на только для файла изображения - *.bmp, *.gif, *.ico, *.jpg, *.png, *.wdp, *.tiff, но и для объекта DrawingImage. 

Для встраивания изображений прямо в сборку .NET как двоичных ресурсов, нужно выбрать файлы в окне Обозреватель решения и установить свойства: "действие при сборке" - "Внедренный ресурс", свойство "Копировать" - "Не копировать". Графические данные больше не копируются в выходную папку, а встраиваются в саму сборку. Это обеспечивает ресурсы, но увеличивает размер скомпилированной сборки.

Пример:

```csharp
...
    Loaded="MainWindow_OnLoaded" Title="MainWindow" Height="400" Width="600" WindowStartupLocation="CenterScreen">
<DockPanel LastChildFill="True">
    <ToolBar Height="60" x:Name="picturePickerToolbar" DockPanel.Dock="Top">
        <Button x:Name="buttonPreviousImage" Height="30" Width="100" BorderBrush="Black" Margin="5" Content="Предидущее" Click="ButtonPreviousImage_OnClick"/>
        <Button x:Name="buttonNextImage" Height="30" Width="100" BorderBrush="Black" Margin="5" Content="Следущее" Click="ButtonNextImage_OnClick"/>
    </ToolBar>
    <Border BorderThickness="2" BorderBrush="LightGreen">
        <Image x:Name="imageHolder" Stretch="Fill"/>
    </Border>
</DockPanel>
private List<BitmapImage> images = new List<BitmapImage>();
private int currentImage = 0;
public MainWindow()
{
    InitializeComponent();

}
private void MainWindow_OnLoaded(object sender, RoutedEventArgs e)
{
    try
    {
        //string path = Environment.CurrentDirectory;
        //загрузка изображений из файлов
        //images.Add(new BitmapImage(new Uri($@"{path}\ma0.jpg")));
        //images.Add(new BitmapImage(new Uri($@"{path}\Images\ma1.jpg")));
        //извлечь из сборки и затем загрузить изображения
        images.Add(new BitmapImage(new Uri($@"Images/ma0.jpg", UriKind.Relative)));
        images.Add(new BitmapImage(new Uri($@"Images/ma1.jpg", UriKind.Relative)));
        images.Add(new BitmapImage(new Uri($@"Images/ma2.jpg", UriKind.Relative)));
        imageHolder.Source = images[currentImage]; //первое изображение в списке
    }
    catch (Exception ex)
    {
        MessageBox.Show(ex.Message);
    }
}
private void ButtonPreviousImage_OnClick(object sender, RoutedEventArgs e)
{
    if (--currentImage < 0)
        currentImage = images.Count - 1;
    imageHolder.Source = images[currentImage];
}
private void ButtonNextImage_OnClick(object sender, RoutedEventArgs e)
{
    if (++currentImage >= images.Count)
        currentImage = 0;
    imageHolder.Source = images[currentImage];
}
```

## Объектные ресурсы

Объектные ресурсы позволяют определить фрагмент разметки XAML, назначить ему имя и сохранить в подхожящем словаре для использования в будущем. Они компилируются в сборку, где они требуются. При условии, что разметка XAML помещена в корректное местоположение, компилятор позаботится обо всем остальном. Это может быть не только кисти, но и анимация, трехмерная визуализация, стили, шаблон данных, шаблон элемента управления и др.

Каждый объект, производный от FrameworkElement содержит свойство Resources, которое инкапсулирует объект ResourceDictionary. Этот объект может храниь элементы любого типа, т.к. тип - System.Object. Все обычные элементы XAML и плюс класс Appplication поддерживают свойство Resources.

Атрибут x:Name позволяет получить доступ к объекту как к переменной-члену в файле кода, а x:Key - к ресурсу в словаре ресурсов. 

Расширение разметки {StaticResource} применяет ресурс только один раз и остается подключенным к первоначальному ресурсу всю работу приложения. Хотя некоторые свойства - такие как градиентные переходы - будут обновлятся.

Расширение разметки {DynamicResource} способно обнаруживать замену внутреннего объекта, указанного посредством ключа, новым объектом, но это требует дополнительной инфраструктуры времени выполнения.

### Ресурсы уровня окна.

Пример:

```csharp
...
    Title="MainWindow" Height="400" Width="600" WindowStartupLocation="CenterScreen">
<Window.Resources>
    <RadialGradientBrush x:Key="myBrush">
        <GradientStop Color="Aqua" Offset="0"/>
        <GradientStop Color="Azure" Offset="0.7"/>
        <GradientStop Color="Blue" Offset="1"/>
    </RadialGradientBrush>
</Window.Resources>
<StackPanel Orientation="Horizontal">
    <Button Margin="10" Height="200" Width="200" Content="OK" FontSize="20" Background="{StaticResource myBrush}"/>
    <Button Margin="10" Height="200" Width="200" Content="Отмена" FontSize="20" Background="{StaticResource myBrush}"/>
</StackPanel>
...
```

Пример изменения ресурса:

```csharp
...
    Title="MainWindow" Height="400" Width="600" WindowStartupLocation="CenterScreen">
    <Window.Resources>
        <RadialGradientBrush x:Key="myBrush">
            <GradientStop Color="Aqua" Offset="0"/>
            <GradientStop Color="Azure" Offset="0.7"/>
            <GradientStop Color="Blue" Offset="1"/>
        </RadialGradientBrush>
    </Window.Resources>
    <StackPanel Orientation="Horizontal">
        <Button Margin="10" Height="200" Width="200" Content="OK" FontSize="20" Background="{StaticResource myBrush}" Click="ButtonOk_OnClick"/>
        <Button Margin="10" Height="200" Width="200" Content="Отмена" FontSize="20" Background="{DynamicResource myBrush}" Click="ButtonCancel_OnClick"/>
    </StackPanel>
</Window>
private void ButtonOk_OnClick(object sender, RoutedEventArgs e)
{
    //var brush = (RadialGradientBrush)Resources["myBrush"];
    var brush = (RadialGradientBrush)TryFindResource("myBrush");
    if (brush == null)
        return;
    brush.GradientStops[0] = new GradientStop(Colors.Black, 0.5); //это сработает
}
private void ButtonCancel_OnClick(object sender, RoutedEventArgs e)
{
    Resources["myBrush"] = new SolidColorBrush(Colors.Green); //а это не сработает
}
```

### Ресурсы уровня приложения.

Пример объявления ресурсов уровня приложения:

```csharp
<Application x:Class="WpfApp1.App"
...
             >
    <Application.Resources>
        <RadialGradientBrush x:Key="myBrush">
            <GradientStop Color="Aqua" Offset="0"/>
            <GradientStop Color="Azure" Offset="0.7"/>
            <GradientStop Color="Blue" Offset="1"/>
        </RadialGradientBrush>
    </Application.Resources>
</Application>
```

Пример объединанный словарь:

Словарь - файл "Dictionary1.xaml":

```csharp
<ResourceDictionary xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
                    xmlns:local="clr-namespace:ResourcesApp">
    <RadialGradientBrush x:Key="myBrush">
        <GradientStop Color="Aqua" Offset="0"/>
        <GradientStop Color="Azure" Offset="0.7"/>
        <GradientStop Color="Blue" Offset="1"/>
    </RadialGradientBrush>
</ResourceDictionary>
```

Использование:

```csharp
<Application x:Class="WpfApp1.App"
...
             >
    <Application.Resources>        
        <ResourceDictionary>
            <ResourceDictionary.MergedDictionaries>
                <ResourceDictionary Source="Dictionary1.xaml" />
            </ResourceDictionary.MergedDictionaries>
        </ResourceDictionary>
    </Application.Resources>
</Application>
```

### Словать ресурсов

Объединенный словать ресурсов - библиотека классов для ресурсов WPF - файл XAML, содержащий коллекцию ресурсов. Проект может содержать любое количество файлов ресурсов.

Пример словаря ресурсов:

```csharp
<ResourceDictionary xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
...
                    xmlns:local="clr-namespace:WpfApp1">
    <RadialGradientBrush x:Key="myBrush">
        <GradientStop Color="Aqua" Offset="0"/>
        <GradientStop Color="Azure" Offset="0.7"/>
        <GradientStop Color="Blue" Offset="1"/>
    </RadialGradientBrush>
</ResourceDictionary>
//подключение файла ресурсов:
<Application.Resources>
    <ResourceDictionary>
        <ResourceDictionary.MergedDictionaries>
            <ResourceDictionary Source="MyBrushes.xaml"/>
        </ResourceDictionary.MergedDictionaries>
    </ResourceDictionary>
</Application.Resources>
```






