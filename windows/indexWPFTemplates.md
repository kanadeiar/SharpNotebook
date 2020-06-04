# WPF Шаблоны (логические, визуальные деревья, стандартные шаблоны, + стили)

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

Пример просмотра стандартного шаблона элемента управления:
```csharp
<StackPanel>
    <Label Content="Введите имя WPF компонента" Width="300" FontWeight="DemiBold"/>
    <TextBox x:Name="textFullName" Width="300" BorderBrush="Green" Background="BlanchedAlmond" Height="22" Text="System.Windows.Controls.Button"/>
    <Button x:Name="buttonTemplate" Content="Просмотреть шаблон" BorderBrush="Green" Height="30" Width="150" Margin="5" HorizontalAlignment="Left" Click="ButtonTemplate_OnClick"/>
    <Border BorderBrush="DarkGreen" BorderThickness="2" Height="280" Width="280" Margin="10" Background="LightGreen">
        <StackPanel x:Name="panelTemplate"></StackPanel>
    </Border>
</StackPanel>
private Control controlToShow = null;
private void ButtonTemplate_OnClick(object sender, RoutedEventArgs e)
{
    string dataToShow = "";
    if (controlToShow != null) //удалить элемент в области просмотра
        panelTemplate.Children.Remove(controlToShow);
    try
    {
        Assembly asm = Assembly.Load("PresentationFramework, Version=4.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35" );
        controlToShow = (Control) asm.CreateInstance(textFullName.Text);
        controlToShow.Height = 200;
        controlToShow.Width = 200;
        controlToShow.Margin = new Thickness(5);
        panelTemplate.Children.Add(controlToShow);
        var xmlSettings = new XmlWriterSettings {Indent = true};
        var strBuilder = new StringBuilder();
        var xWriter = XmlWriter.Create(strBuilder, xmlSettings);
        XamlWriter.Save(controlToShow, xWriter);
        dataToShow = strBuilder.ToString();
    }
    catch (Exception ex)
    {
        dataToShow = ex.Message;
    }
    textBoxDisplayArea.Text = dataToShow;
}
```

## Построение шаблона

Построение специальных шаблонов производится легче в Blend, чем напрямую в Visual Studio. При построении специальных шаблонов следует помнить, что поведение элемента управления неизменно, но его внешний вид может варьироваться. Определение собственного шаблона сводится к замене стандартного визуального дерева своим вариантом. 

Если шаблон использует какой-либо объект нестандартной формы для отображения необычной геометрии, тогда можно иметь уверенность в том, что детали проверки попадания курсора будут соответствовать форме элемента управления, а не более крупного ограничивающего прямоугольника.

Пример определения кнопки необычной геометрии:
```csharp
<StackPanel>
    <Button x:Name="buttonMy" Width="100" Height="100" Click="ButtonMy_OnClick">
        <Button.Template>
            <ControlTemplate>
                <Grid x:Name="controlLayout">
                    <Ellipse x:Name="buttonSurfce" Fill="LightPink"/>
                    <Label x:Name="buttonCaption" VerticalAlignment="Center" HorizontalAlignment="Center" FontWeight="Bold" FontSize="24" Content="OK!"/>
                </Grid>
            </ControlTemplate>
        </Button.Template>
    </Button>
</StackPanel>
private void ButtonMy_OnClick(object sender, RoutedEventArgs e)
{
    MessageBox.Show("авыаыв");
}
```
Пример переноса ресурса-кнопки в файл App.xaml и его использование сразу в нескольких кнопках:
```csharp
<Application.Resources>
    <ControlTemplate x:Key="TemplateRoundButton" TargetType="{x:Type Button}">
        <Grid x:Name="controlLayout">
            <Ellipse x:Name="buttonSurfce" Fill="LightPink"/>
            <Label x:Name="buttonCaption" VerticalAlignment="Center" HorizontalAlignment="Center" FontWeight="Bold" FontSize="24" Content="OK!"/>
        </Grid>
    </ControlTemplate>
</Application.Resources>
<StackPanel>
    <Button x:Name="buttonMy1" Margin="5" Width="100" Height="100" Click="ButtonMy_OnClick" Template="{StaticResource TemplateRoundButton}"/>
    <Button x:Name="buttonMy2" Margin="5" Width="50" Height="50" Template="{StaticResource TemplateRoundButton}"/>
    <Button x:Name="buttonMy3" Margin="5" Width="150" Height="150" Template="{StaticResource TemplateRoundButton}"/>
</StackPanel>
private void ButtonMy_OnClick(object sender, RoutedEventArgs e)
{
    MessageBox.Show("авыаыв");
}
```
Стандартный шаблон кнопки сожержит разметку, которая задает внешний вид управления при возникновении определенных событий пользоватлеьского интерфейса - таких как фокус, щелчок кнопкой мыши, включение или отключение. Пользователи хорошо приучены к визуальным элементам, дающим элементу управления осязаемую реакцию. Когда совершаются определенные действия пользователя, элемент должен на это реагировать.

Пример шаблона с добавленой разметкой с триггерами, которые будут реагировать на действия пользователя.
```csharp
<Application.Resources>
    <ControlTemplate x:Key="TemplateRoundButton" TargetType="{x:Type Button}">
        <Grid x:Name="controlLayout">
            <Ellipse x:Name="buttonSurfce" Fill="LightPink"/>
            <Label x:Name="buttonCaption" VerticalAlignment="Center" HorizontalAlignment="Center" FontWeight="Bold" FontSize="24" Content="OK!"/>
        </Grid>
        <ControlTemplate.Triggers>
            <Trigger Property="IsMouseOver" Value="True">
                <Setter TargetName="buttonSurfce" Property="Fill" Value="Blue"/>
                <Setter TargetName="buttonCaption" Property="Foreground" Value="Yellow"/>
            </Trigger>
            <Trigger Property="IsPressed" Value="True">
                <Setter TargetName="controlLayout" Property="RenderTransformOrigin" Value="0.5,0.5"/>
                <Setter TargetName="controlLayout" Property="RenderTransform">
                    <Setter.Value>
                        <ScaleTransform ScaleX="0.8" ScaleY="0.8"/>
                    </Setter.Value>
                </Setter>
            </Trigger>
        </ControlTemplate.Triggers>
    </ControlTemplate>
</Application.Resources>
```

Расширение разметки {TemplateBinding} при построении шаблона позволяет захватывать настройки свойств, которые определены элементом управления, применяющим шаблон.

Пример примененеия шаблона с расширением разметки {TemplateBinding}:
```csharp
<Application.Resources>
    <ControlTemplate x:Key="TemplateRoundButton" TargetType="{x:Type Button}">
        <Grid x:Name="controlLayout">
            <Ellipse x:Name="buttonSurfce" Fill="{TemplateBinding Background}"/>
            <Label x:Name="buttonCaption" Content="{TemplateBinding Content}" VerticalAlignment="Center" HorizontalAlignment="Center" FontWeight="Bold" FontSize="24"/>
        </Grid>
    </ControlTemplate>
</Application.Resources>
<StackPanel>
    <Button x:Name="buttonMy1" Margin="5" Width="100" Height="100" Background="Red" Content="Go!" Click="ButtonMy_OnClick" Template="{StaticResource TemplateRoundButton}"/>
    <Button x:Name="buttonMy2" Margin="5" Width="50" Height="50" Background="Aqua" Content="Hi!" Template="{StaticResource TemplateRoundButton}"/>
    <Button x:Name="buttonMy3" Margin="5" Width="150" Height="150" Background="Bisque" Content="Haha" Template="{StaticResource TemplateRoundButton}"/>
</StackPanel>
```

Класс ContentPresenter позволяет определить обобщенную область отображения содержимого в шаблоне, при этом содержимое берется из свойства Content элемента управления, использующего шаблон.

Пример использования этого класса:
```csharp
<Application.Resources>
    <ControlTemplate x:Key="NewTemplateRoundButton" TargetType="{x:Type Button}">
        <Grid>
            <Ellipse Fill="{TemplateBinding Background}"/>
            <ContentPresenter HorizontalAlignment="Center" VerticalAlignment="Center"/>
        </Grid>
    </ControlTemplate>
</Application.Resources>
<Button x:Name="buttonMy4" Margin="5" Width="150" Height="50" Background="LightGray" Template="{StaticResource NewTemplateRoundButton}">
    <StackPanel Orientation="Horizontal">
        <Label FontSize="30" Content="@"></Label>
        <Label FontSize="23" Content="123"></Label>
    </StackPanel>
</Button>
```

## Шаблоны + стили

При построении стиля можно определить шаблон внутри стиля. При этом появляется возможность предоставить готовый набор значений для общих свойств.

Пример определения стиля с шаблоном и его применения:
```csharp
<Application.Resources>
    <Style x:Key="RoundButtonStyle" TargetType="Button">
        <Setter Property="Foreground" Value="BlueViolet"/>
        <Setter Property="FontSize" Value="14"/>
        <Setter Property="FontWeight" Value="Bold"/>
        <Setter Property="Width" Value="100"/>
        <Setter Property="Height" Value="100"/>
        <Setter Property="Template">
            <Setter.Value>
                <ControlTemplate TargetType="Button">
                    <Grid x:Name="controlLayout">
                        <Ellipse Fill="{TemplateBinding Background}"/>
                        <Label Content="{TemplateBinding Content}" VerticalAlignment="Center" HorizontalAlignment="Center" FontWeight="Bold" FontSize="24" Foreground="Yellow"/>
                    </Grid>
                </ControlTemplate>
            </Setter.Value>
        </Setter>
    </Style>
</Application.Resources>
<StackPanel>
    <Button x:Name="buttonBestMy" Background="Red" Content="Hia!" Click="ButtonBestMy_OnClick" Style="{StaticResource RoundButtonStyle}">
    </Button>
</StackPanel>
```




