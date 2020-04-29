# Основы WPF Триггеры

## Триггеры свойств

Пример триггеров свойств:
```csharp
<Window x:Class="WpfApp1HelloWPF.MainWindow"
...
        Title="Заголовок окна" Height="400" Width="600" WindowStartupLocation="CenterScreen">
    <Grid Margin="10">
        <TextBlock Text="Триггер" FontSize="28" HorizontalAlignment="Center" VerticalAlignment="Center">
            <TextBlock.Style>
                <Style TargetType="TextBlock">
                    <Setter Property="Foreground" Value="Blue"></Setter>
                    <Style.Triggers>
                        <Trigger Property="IsMouseOver" Value="True">
                            <Setter Property="Foreground" Value="Red"/>
                            <Setter Property="TextDecorations" Value="Underline"/>
                        </Trigger>
                    </Style.Triggers>
                </Style>
            </TextBlock.Style>
        </TextBlock>
    </Grid>
</Window>
```

## Триггеры данных

Пример триггеров данных:
```csharp
<Window x:Class="WpfApp1HelloWPF.MainWindow"
...
        Title="Заголовок окна" Height="400" Width="600" WindowStartupLocation="CenterScreen">
    <StackPanel HorizontalAlignment="Center" VerticalAlignment="Center">
        <CheckBox Name="checkBoxSimple" Content="Отметь меня"/>
        <TextBlock HorizontalAlignment="Center" Margin="0,20,0,0" FontSize="48">
            <TextBlock.Style>
                <Style TargetType="TextBlock">
                    <Setter Property="Text" Value="Нет"/>
                    <Setter Property="Foreground" Value="Red"/>
                    <Style.Triggers>
                        <DataTrigger Binding="{Binding ElementName=checkBoxSimple, Path=IsChecked}" Value="True">
                            <Setter Property="Text" Value="Да"/>
                            <Setter Property="Foreground" Value="Green"/>
                        </DataTrigger>
                    </Style.Triggers>
                </Style>
            </TextBlock.Style>
        </TextBlock>
    </StackPanel>
</Window>
```

## Триггеры событий

Примеры триггеров событий для анимации:
```csharp
<Window x:Class="WpfApp1HelloWPF.MainWindow"
...
        Title="Заголовок окна" Height="400" Width="600" WindowStartupLocation="CenterScreen">
    <Grid>
        <TextBlock Name="textBlockLabel" Text="EventTrigger" FontSize="18" HorizontalAlignment="Center" VerticalAlignment="Center">
            <TextBlock.Style>
                <Style TargetType="TextBlock">
                    <Style.Triggers>
                        <EventTrigger RoutedEvent="MouseEnter">
                            <EventTrigger.Actions>
                                <BeginStoryboard>
                                    <Storyboard>
                                        <DoubleAnimation Duration="0:0:0.900" Storyboard.TargetProperty="FontSize" To="28"/>
                                    </Storyboard>
                                </BeginStoryboard>
                            </EventTrigger.Actions>
                        </EventTrigger>
                        <EventTrigger RoutedEvent="MouseLeave">
                            <EventTrigger.Actions>
                                <BeginStoryboard>
                                    <Storyboard>
                                        <DoubleAnimation Duration="0:0:0.900" Storyboard.TargetProperty="FontSize" To="16"/>
                                    </Storyboard>
                                </BeginStoryboard>
                            </EventTrigger.Actions>
                        </EventTrigger>
                    </Style.Triggers>
                </Style>
            </TextBlock.Style>
        </TextBlock>
    </Grid>
</Window>
```






