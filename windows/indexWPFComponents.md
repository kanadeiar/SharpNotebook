# WPF Элементы (примеры элементов)

Документация о конкретной функциональности заданного элемента управления дает документация .NET Framework 4.7 SDK - раздел ["Библиотека элементов управления"](https://docs.microsoft.com/ru-ru/dotnet/framework/wpf/controls/). справочной системы.

Некоторые элементы управления WPF:
```csharp
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
```

WPF предоставляет элементы для работы с документами в стиле Adobe PDF, с применением типов из пространства имен System.Windows.Documents (PresentationFramework.dll), работающие с API-интерфейсом XML Paper Specification (XPS).

WPF Предлагает несколько общих диалоговых окон, как OpenFilaDialog и SaveFileDialog, определенные в пространстве имен Windows.Win32 внутри PresentationFramework.dll.

Пример диалогового окна:
```csharp
private void MyButton_OnClick(object sender, RoutedEventArgs e)
{
    SaveFileDialog saveDlg = new SaveFileDialog();
    saveDlg.ShowDialog();
}
```



## TextBlock
```csharp
<Window x:Class="WpfApp1HelloWPF.MainWindow"
...
        Title="Заголовок окна" Height="600" Width="800">
    <StackPanel Margin="10">
        <TextBlock Margin="10" Foreground="Red">Текстовый блок</TextBlock>
        <TextBlock Margin="10" TextTrimming="CharacterEllipsis" Foreground="Green">Текстовый блок</TextBlock>
        <TextBlock Margin="10" TextWrapping="Wrap" Foreground="Blue">Текстовый блок</TextBlock>
    </StackPanel>
</Window>
```

## Label
```csharp
<Window x:Class="WpfApp1HelloWPF.MainWindow"
...
        Title="Заголовок окна" Height="600" Width="800">
    <StackPanel Margin="10">
        <Label Content="Текстовая надпись" FontSize="28"/>
    </StackPanel>
</Window>
```

## TextBox
```csharp
<Window x:Class="WpfApp1HelloWPF.MainWindow"
...
        Title="Заголовок окна" Height="600" Width="800">
    <StackPanel Margin="10">
        <TextBox AcceptsReturn="True" TextWrapping="Wrap" FontSize="28"/>
    </StackPanel>
</Window>
```

## Button
```csharp
<Window x:Class="WpfApp1HelloWPF.MainWindow"
...
        Title="Заголовок окна" Height="600" Width="800">
    <StackPanel Margin="10">
        <TextBox AcceptsReturn="True" TextWrapping="Wrap" FontSize="28"/>
        <Button x:Name="buttonClickMe" Click="buttonClickMe_Click" Content="Нажми меня" FontSize="28"/>
    </StackPanel>
</Window>
private void buttonClickMe_Click(object sender, RoutedEventArgs e)
{
    MessageBox.Show("Клавиша нажата!");
}
```

## CheckBox
```csharp
<Window x:Class="WpfApp1HelloWPF.MainWindow"
...
        Title="Заголовок окна" Height="450" Width="600" WindowStartupLocation="CenterScreen">
    <StackPanel Margin="10">
        <CheckBox x:Name="checkBox1" IsThreeState="True" IsChecked="False" Height="20" Content="Не отмечено"/>
        <CheckBox x:Name="checkBox2" IsThreeState="True" IsChecked="True" Height="20" Content="Отмечено"/>
        <CheckBox x:Name="checkBox3" IsThreeState="True" IsChecked="{x:Null}" Height="20" Content="Неопределено" Unchecked="checkBox3_Unchecked" Indeterminate="checkBox3_Indeterminate" Checked="checkBox3_Checked"/>
    </StackPanel>
</Window>
private void checkBox3_Unchecked(object sender, RoutedEventArgs e)
{
    MessageBox.Show("Не отмечен");
}
private void checkBox3_Indeterminate(object sender, RoutedEventArgs e)
{
    MessageBox.Show("В неопределенном состоянии");
}
private void checkBox3_Checked(object sender, RoutedEventArgs e)
{
    MessageBox.Show("Отмечен");
}
```

## RadioButton
```csharp
<Window x:Class="WpfApp1HelloWPF.MainWindow"
...
        Title="Заголовок окна" Height="450" Width="600" WindowStartupLocation="CenterScreen">
    <StackPanel Margin="10">
        <Label FontWeight="Bold">Ответ:</Label>
        <RadioButton GroupName="ready">Да</RadioButton>
        <RadioButton GroupName="ready" IsChecked="True">Нет</RadioButton>
    </StackPanel>
</Window>
```

## ItemsControl
```csharp
<Window x:Class="WpfApp1HelloWPF.MainWindow"
...
        Title="Заголовок окна" Height="450" Width="600" WindowStartupLocation="CenterScreen">
    <StackPanel Margin="10">
        <ItemsControl>
            <sys:String>Строка 1</sys:String>
            <sys:String>Строка 2</sys:String>
            <sys:String>Строка 3</sys:String>
        </ItemsControl>
    </StackPanel>
</Window>
```

## ListBox
```csharp
<Window x:Class="WpfApp1HelloWPF.MainWindow"
...
        Title="Заголовок окна" Height="400" Width="600" WindowStartupLocation="CenterScreen">
    <Grid>
        <ListBox Margin="10" Name="listBoxEmp" SelectionMode="Single" VerticalAlignment="Top" Height="250" SelectionChanged="listBoxEmp_SelectionChanged"/>
        <Button Margin="10" Height="30" Width="100" VerticalAlignment="Bottom" Click="Button_Click">Добавить</Button>
    </Grid>
</Window>
public partial class MainWindow : Window
{
    ObservableCollection<Employe> items = new ObservableCollection<Employe>()
    {
        new Employe{Id=1,Name="Vasya",Age=22,Salary=3000},
        new Employe{Id=2,Name="Ivan",Age=25,Salary=6000},
        new Employe{Id=3,Name="Kolya",Age=23,Salary=8000},
    };
    public MainWindow()
    {
        InitializeComponent();
        listBoxEmp.ItemsSource = items;
    }
    private void listBoxEmp_SelectionChanged(object sender, SelectionChangedEventArgs e)
    {
        MessageBox.Show(e.Source.ToString());
    }
    private void Button_Click(object sender, RoutedEventArgs e)
    {
        items.Add(new Employe{Id=9,Name="New",Age=30,Salary=9000});
    }
}
public class Employe
{
    public int Id {get;set;}
    public string Name {get;set;}
    public int Age {get;set;}
    public double Salary {get;set;}
    public override string ToString() => $"{Id}\t{Name}\t{Age}\t{Salary}";
}
```

## ComboBox
```csharp
<Window x:Class="WpfApp1HelloWPF.MainWindow"
...
WindowStartupLocation="CenterScreen">
    <Grid>
        <ComboBox x:Name="comboBox" HorizontalAlignment="Center" VerticalAlignment="Top" Width="200" SelectionChanged="comboBox_SelectionChanged">
            <ComboBoxItem>Элемент 1</ComboBoxItem>
            <ComboBoxItem>Элемент 2</ComboBoxItem>
            <ComboBoxItem>Элемент 3</ComboBoxItem>
        </ComboBox>
    </Grid>
</Window>
private void comboBox_SelectionChanged(object sender, SelectionChangedEventArgs e)
{
    ComboBox comboBox = (ComboBox)sender;
    ComboBoxItem selectedItem = (ComboBoxItem)comboBox.SelectedItem;
    MessageBox.Show(selectedItem.Content?.ToString());
}
```

## Window

Дочерние окна.
```csharp
<Window x:Class="WpfApp1HelloWPF.MainWindow"
...
WindowStartupLocation="CenterScreen">
    <Grid>
        <Button Width="100" Height="30" Content="Дочернее окно" Click="Button_Click" Margin="10,10,200,10"/>
        <Button Width="100" Height="30" Content="Цвет всех окон" Click="Button_Click_1" Margin="200,10,10,10"/>
    </Grid>
</Window>
private void Button_Click(object sender, RoutedEventArgs e)
{
    ChildWindow childWindow = new ChildWindow();
    childWindow.ViewModel = "Дочернее окно";
    childWindow.Owner = Window.GetWindow(this);;
    childWindow.Show();
    childWindow.ShowViewModel();
    foreach (Window win in this.OwnedWindows)
    {
        win.Background = new SolidColorBrush(Colors.Aquamarine);
        if (win is ChildWindow)
            win.Title = "Новый заголовок";
    }
}
private void Button_Click_1(object sender, RoutedEventArgs e)
{
    foreach (Window win in App.Current.Windows)
    {
        win.Background = new SolidColorBrush(Colors.LightGreen);
    }
}
//дочернее окно:
<Window x:Class="WpfApp1HelloWPF.ChildWindow"
...
        Title="ChildWindow" Height="400" Width="600">
    <Grid>
        <TextBlock x:Name="textBlock" HorizontalAlignment="Center" TextWrapping="Wrap" Text="TextBlock" VerticalAlignment="Center" FontSize="28"/>
        <Button x:Name="myButton" Height="30" Width="100" VerticalAlignment="Top" Content="Кнопка" Click="myButton_Click"/>
    </Grid>
</Window>
public string ViewModel {get;set;}
public void ShowViewModel()
{
    textBlock.Text = ViewModel;
}
private void myButton_Click(object sender, RoutedEventArgs e)
{
    this.Owner.Title = "Измененный заголовок";
}
```

## ListView

Компонент отображающий данные из списка:

```csharp
<Window x:Class="WpfApp1HelloWPF.MainWindow"
...
        Title="Заголовок окна" Height="400" Width="600" WindowStartupLocation="CenterScreen">
    <Grid>
        <ListView Margin="10" Name="listViewEmployee">
            <ListView.View>
                <GridView>
                    <GridViewColumn Header="Имя" Width="220" DisplayMemberBinding="{Binding Name}"/>
                    <GridViewColumn Header="Возраст" Width="80" DisplayMemberBinding="{Binding Age}"/>
                    <GridViewColumn Header="Зарплата" Width="150">
                        <GridViewColumn.CellTemplate>
                            <DataTemplate>
                                <TextBlock Text="{Binding Salary}" Foreground="Blue" FontWeight="Bold"/>
                            </DataTemplate>
                        </GridViewColumn.CellTemplate>
                    </GridViewColumn>
                </GridView>
            </ListView.View>
        </ListView>
    </Grid>
</Window>
public partial class MainWindow : Window
{
    public MainWindow()
    {
        InitializeComponent();
        List<Employee> items = new List<Employee>
        {
            new Employee {Name="Петя",Age=42,Salary=25000},
            new Employee {Name="Толя",Age=12,Salary=9000},
            new Employee {Name="Боря",Age=22,Salary=11000},
        };
        listViewEmployee.ItemsSource = items;
    }
}
public class Employee
{
    public string Name {get;set;}
    public int Age {get;set;}
    public int Salary {get;set;}
}
```

Виртуализация для ускорения работы компонента при больших данных. Пример:

```csharp
<Window x:Class="WpfApp1HelloWPF.MainWindow"
...
        Title="Заголовок окна" Height="400" Width="600" WindowStartupLocation="CenterScreen">
    <Grid>
        <ComboBox Name="comboBoxNames" Margin="10" VerticalAlignment="Top" VirtualizingStackPanel.IsVirtualizing="True">
            <ComboBox.ItemsPanel>
                <ItemsPanelTemplate>
                    <VirtualizingStackPanel/>
                </ItemsPanelTemplate>
            </ComboBox.ItemsPanel>
        </ComboBox>
    </Grid>
</Window>
public MainWindow()
{
    InitializeComponent();
    List<string> names = new List<string>();
    for (int i=1; i<3000; i++)
        names.Add($"Имя {i-500}");
    comboBoxNames.ItemsSource = names;
}
```

## InkCanvas

Компонент рисования в окне:
```csharp
    <WrapPanel>
        <RadioButton x:Name="inkRadio" Margin="5,10" Content="InkMode" IsChecked="True" Click="RadioButton_Click"/>
        <RadioButton x:Name="eraseRadio" Margin="5,10" Content="Erase Mode" Click="RadioButton_Click"/>
        <RadioButton x:Name="selectRadio" Margin="5,10" Content="SelectMode" Click="RadioButton_Click"/>
    </WrapPanel>
...
<ComboBox x:Name="comboColors" Width="100" SelectionChanged="comboColors_SelectionChanged" Height="30">
    <StackPanel Orientation="Horizontal" Tag="Red">
        <Ellipse Fill="Red" Height="20" Width="20"/>
        <Label Content="Красный"/>
    </StackPanel>
    <StackPanel Orientation="Horizontal" Tag="Green">
        <Ellipse Fill="Green" Height="20" Width="20"/>
        <Label Content="Зеленый"/>
    </StackPanel>
    <StackPanel Orientation="Horizontal" Tag="Blue">
        <Ellipse Fill="Blue" Height="20" Width="20"/>
        <Label Content="Синий"/>
    </StackPanel>
...
    <Button Grid.Column="0" x:Name="buttonSave" Height="30" Width="70" Margin="2" Content="Сохранить" Click="buttonSave_Click"/>
    <Button Grid.Column="1" x:Name="buttonLoad" Height="30" Width="70" Margin="2" Content="Загрузить" Click="buttonLoad_Click"/>
    <Button Grid.Column="2" x:Name="buttonClear" Height="30" Width="70" Margin="2" Content="Очистить" Click="buttonClear_Click"/>
...
public MainWindow()
{
    InitializeComponent();
    myInkCanvas.EditingMode = InkCanvasEditingMode.Ink;
    inkRadio.IsChecked = true;
    this.comboColors.SelectedIndex = 0;
}
private void RadioButton_Click(object sender, RoutedEventArgs e)
{
    switch ((sender as RadioButton)?.Name)
    {
        case "inkRadio":
            myInkCanvas.EditingMode = InkCanvasEditingMode.Ink;
            break;
        case "eraseRadio":
            myInkCanvas.EditingMode = InkCanvasEditingMode.EraseByStroke;
            break;
        case "selectRadio":
            myInkCanvas.EditingMode = InkCanvasEditingMode.Select;
            break;
    }
}
private void comboColors_SelectionChanged(object sender, SelectionChangedEventArgs e)
{
    string color = (comboColors.SelectedItem as StackPanel)?.Tag.ToString();
    myInkCanvas.DefaultDrawingAttributes.Color = (Color)ColorConverter.ConvertFromString(color);
}
private void buttonSave_Click(object sender, RoutedEventArgs e)
{
    using (FileStream stream = new FileStream("Stroke.bin",FileMode.Create,FileAccess.Write))
    {
        myInkCanvas.Strokes.Save(stream);
    }
}
private void buttonLoad_Click(object sender, RoutedEventArgs e)
{
    using (FileStream stream = new FileStream("Stroke.bin",FileMode.Open,FileAccess.Read))
    {
        var strokes = new StrokeCollection(stream);
        myInkCanvas.Strokes = strokes;
    }
}
private void buttonClear_Click(object sender, RoutedEventArgs e)
{
    myInkCanvas.Strokes.Clear();
}
```







