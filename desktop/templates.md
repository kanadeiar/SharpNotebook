# Шаблоны

Логическое представление отражает то, как содержимое будет позиционировано внутри разнообразных диспетчеров компоновки для главного элемента.

За каждым логическим уровнем есть визуаьное дерево для корректной визуализации элементов на экране. Внутри него - полные детали шаблонов и стилей визуализации каждого объекта.

## Стандартный шаблон

Любой шаблон может быть представлен как экземпляр класса ControlTemplate. 

Стандартный шаблон можно получить через свойство Templete элемента управления:

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

## Построение шаблона

Определение собственного шаблона сводится к замене стандартного визуального дерева своим вариантом.

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
    MessageBox.Show("Щелчок по необычной кнопке");
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

Стандартный шаблон кнопки содержит разметку, которая задает внешний вид управления при возникновении определенных событий пользоватлеьского интерфейса - таких как фокус, щелчок кнопкой мыши, включение или отключение. Пользователи хорошо приучены к визуальным элементам, дающим элементу управления осязаемую реакцию. Когда совершаются определенные действия пользователя, элемент должен на это реагировать.

Пример шаблона с добавленой разметкой с триггерами, которые будут реагировать на действия пользователя:

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

Пример применения шаблона с расширением разметки {TemplateBinding}:

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

## Шаблон + стиль

При построении стиля можно определить шаблон внутри стиля. При этом появляется возможность предоставить готовый набор значений для общих свойств.

Пример определения стиля с шаблоном и его применение:

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
    <Button x:Name="buttonBestMy" Background="Red" Click="ButtonMy_OnClick" Style="{StaticResource RoundButtonStyle}">
    Кнопка
    </Button>
</StackPanel>
```



