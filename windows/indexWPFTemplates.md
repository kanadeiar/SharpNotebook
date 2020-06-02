# WPF Шаблоны (логические, визуальные деревья, стандартные шаблоны)







## Логические деревья

Логическое представление отражает то, как содержимое будет позиционировано внутри разнообразных диспетчеров компоновки для главного элемента.

В пространстве имен System.Windows есть класс LogicalTreeHelper для инспектирования структуры логического дерева во время выполнения.

Пример просмотра логического дерева окна приложения:
```csharp
<DockPanel LastChildFill="True">
    <Border Height="50" DockPanel.Dock="Top" BorderBrush="Blue">
        <StackPanel Orientation="Horizontal">
            <Button x:Name="buttonShowLogicalTree" Content="Логическое дерево" Margin="4" BorderBrush="Blue" Height="40" Click="ButtonShowLogicalTree_OnClick"/>
            <Button x:Name="buttonShowVisualTree" Content="Визуальное дерево" BorderBrush="Blue" Height="40" Click="ButtonShowVisualTree_OnClick"></Button>
        </StackPanel>
    </Border>
    <TextBox x:Name="textBoxDisplayArea" Margin="10" Background="AliceBlue" IsReadOnly="True" BorderBrush="Red" VerticalScrollBarVisibility="Auto" HorizontalScrollBarVisibility="Auto"/>
</DockPanel>
private string dataToShow = string.Empty;
private void ButtonShowLogicalTree_OnClick(object sender, RoutedEventArgs e)
{
    dataToShow = "";
    BuildLogicalTree(0, this);
    textBoxDisplayArea.Text = dataToShow;
}
void BuildLogicalTree(int depth, object obj)
{
    dataToShow += new string(' ', depth) + obj.GetType().Name + "\n";
    if (!(obj is DependencyObject))
        return;
    foreach (var child in LogicalTreeHelper.GetChildren((DependencyObject)obj))
    {
        BuildLogicalTree(depth + 5, child);
    }
}
```

## Визуальные деревья

За каждым логическим уровнем есть визуаьное дерево для корректной визуализации элементов на экране. Внутри него - полные детали шаблонов и стилей визуализации каждого объекта.

Пример просмотра визуального дерева окна приложения:
```csharp
<DockPanel LastChildFill="True">
//тоже самое
</DockPanel>
private string dataToShow = string.Empty;
private void ButtonShowVisualTree_OnClick(object sender, RoutedEventArgs e)
{
    dataToShow = "";
    BuildVisualTree(0, this);
    textBoxDisplayArea.Text = dataToShow;
}
void BuildVisualTree(int depth, DependencyObject obj)
{
    dataToShow += new string(' ',depth) + obj.GetType().Name + "\n";
    for (int i = 0; i < VisualTreeHelper.GetChildrenCount(obj); i++)
    {
        BuildVisualTree(depth + 1, VisualTreeHelper.GetChild(obj, i));
    }
}    
```

## Стандартные шаблоны

Любой шаблон может быть представлен как экземпляр класса ControlTemplate. Стандартный шаблон можно получить через свойство Templete элемента управления:
```csharp
Button myButton = new Button();
ControlTemplate template = myButton.Template; 
```
Можно и сделать обратное - установить шаблон в коде элементу управления:
```csharp
Button myButton = new Button();
ControlTemplate myTemplate = new ControlTemplate();
myButton.Template = myTemplate;
```














