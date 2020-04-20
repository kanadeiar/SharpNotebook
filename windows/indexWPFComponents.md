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

```





