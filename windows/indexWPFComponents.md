# Основы WPF Элементы управления

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
    childWindow.ViewModel = "Дчернее окно";
    childWindow.Owner = this;
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






