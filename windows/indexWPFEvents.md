# WPF События (команды, маршрутизация)

## Команды 

Команды WPF являются похожимы на события сущностями, которые не зависят от специфического элемента управления и может применятся к различным типам элементов управления. WPF поддерживает команды копирования, вырезания и вставки, используемые в разнообразных элементах пользовательского интерфейса, а также клавиатурные комбинации - Ctrl+... . В WPF команда - это любой объект, поддерживающий свойство, возвращающий объект, реализующий интерфейс ICommand.

Команда WPF состоит из четерых элементов:

- сама команда, представляет собой выполняемую задачу

- привязка команд, - связь команды с определенной логикой приложения

- источник команды = элемент пользовательского интерфейса, который запускает команду

- цель команды - элемент интерфейса, на котором выполняется команда

Интерфейс ICommand:
```csharp
public interface ICommand
{
    //возникает, когда изменяется сосотояние команды
    event EventHandler CanExecuteChanged;
    //поределяет метод, который выясняет, может ли команда выполняться в ее текущем состоянии, возвращающий true, если команда включена и доступна для использования
    bool CanExecute(object parameter);
    //определяет метод для хранения логики команды
    void Execute(object parameter);
}
```

Некоторые стандартные объекты команд некоторых классов:
```csharp
Класс ApplicationCommands:
Close, Copy, Cut, Delete, Find, Open, Paste, Save, SaveAs, Redo, Undo
Разнообразные команды уровня приложения
Класс ComponentCommands:
MoveDown, MoveFocusBack, MoveLeft, MoveRight, ScrollToEnd, ScrollToTime
Разнообразные команды, являющимися общими для компонентов пользовательского интерфейса
Класс MediaCommands
BoostBase, ChannelUp, ChannelDown, FastForeard, NextTrack, Play, Rewind, Select, Stop
Разнообразные мультимедийные команды
Класс NavigateCommands
BrowseBack, BrowseForward, Favorites, LastPage, NextPage, Zoom
Разнообразные команды навигационной модели WPF
Класс EditingCommands
AlignCenter, CorrectSpellingError, DecreaseFontSize, EnterLineBreak, EnterParagraphBreak, MoveDownByLine, MoveRightByWord
Разнообразные команды интерфейса Documents API WPF
```

Применение стандартных команд:
```csharp
<MenuItem Header="Редактирование">
    <MenuItem Command="ApplicationCommands.Copy"/>
    <MenuItem Command="ApplicationCommands.Cut"/>
    <MenuItem Command="ApplicationCommands.Paste"/>
</MenuItem>
```

## Команда связанная с произвольным действием

Привязку команд предоставляет объект CommandBinding.

Подключение объекта команд к произвольному событию (специфичному для приложения) производится написанием определенной логики. Необзодимы два обработчика события - первый "CanExecute(object sender, CanExecuteRoutedEventArgs e)" - указывает, индицируется ли команда эта для конкретной операции программы или нет, и второй "Executed(object sender, ExecutedRoutedEventArgs e)" - код, который должен быть выполнен после того, как команда произошла. Они должны быть привязаны к объекту CommandBuilding, реализующему интерфейс ICommand. В конце добавлен в коллекцию команд визуального элемента CommandBindings.

Пример привязки команды к клавише <F1>:
```csharp
public MainWindow()
{
    InitializeComponent();
    CommandBinding commandBinding = new CommandBinding(); //создание объекта-привязки команды
    commandBinding.Command = ApplicationCommands.Help; //установка команды
    commandBinding.Executed += HelpExecuted; //установка метода, выполняемого при вызове команды
    CommandBindings.Add(commandBinding); //добавить привязку к коллекции привязок
}
private void HelpExecuted(object sender, ExecutedRoutedEventArgs e)
{
    MessageBox.Show("Помощь в приложении", "Помощь");
}
```
Эта же привязка но в XAML к кнопке:
```csharp
<Button x:Name="helpButton" Command="ApplicationCommands.Help" Height="30">
    <Button.CommandBindings>
        <CommandBinding Command="ApplicationCommands.Help" Executed="HelpExecuted"/>
    </Button.CommandBindings>
</Button>
private void HelpExecuted(object sender, ExecutedRoutedEventArgs e)
{
    MessageBox.Show("Помощь в приложении", "Помощь");
}
```
Еще пример произвольной команды:
```csharp
public MainWindow()
{
    InitializeComponent();
    SetF1CommandBuilding(); //создать новый объект команду
}
private void SetF1CommandBuilding()
{
    CommandBinding helpBinding = new CommandBinding(ApplicationCommands.Help); //настройка объекта-команды на клавишу <F1>
    helpBinding.CanExecute += CanHelpExecute; //указание индикации команды для конкретной операции
    helpBinding.Executed += HelpExecuted; //код, который должен быть выполнен после выполнения команды
    CommandBindings.Add(helpBinding);
}
private void CanHelpExecute(object sender, CanExecuteRoutedEventArgs e)
{
    e.CanExecute = true; //если нужно предотвратить выполнение команды - установить в false
}
private void HelpExecuted(object sender, ExecutedRoutedEventArgs e)
{
    MessageBox.Show("Помощь в приложении", "Помощь");
}
```
Пример создания инициаторов команд и привязки к ним:
```csharp
<Window x:Class="WpfApp1.MainWindow"
...
        Title="MainWindow" Height="400" Width="500">
    <Window.CommandBindings>
        <CommandBinding Command="ApplicationCommands.Open" Executed="OpenCmdExecuted" CanExecute="OpenCmdCanExecute"/>
        <CommandBinding Command="ApplicationCommands.Save" Executed="SaveCmdExecuted" CanExecute="SaveCmdCanExecute"/>
    </Window.CommandBindings>
    <DockPanel Background="LightSteelBlue">
...
            <MenuItem Header="Файл">
                <MenuItem Command="ApplicationCommands.Open"/>
                <MenuItem Command="ApplicationCommands.Save"/>
                <Separator/>
                <MenuItem Header="Выход" Click="MenuItemExit_OnClick" MouseEnter="UIElement_OnMouseEnter" MouseLeave="UIElement_OnMouseLeave"/>
            </MenuItem>
```

## Маршрутизация событий

Модель марштуризируемых событий является усовершенствованием стандартной модели CLR и сделана для того, чтоб дать возможность обработки событий в манере, подходящей описанию XAML дерева объекта. Марштуризируемые события WPF автоматически вызывают единственный обработчик события компонента интерфейса WPF приложения, вне зависимости от того, на какой части компонента был произведен щелчок.

Если событие перемещается от точки возникновения вверх к другим элементам по дереву объектов, то оно - пузырьковое событие. Если перемещается от самого внешнего вниз к точке возникновения - туннельное событие. Если событие инициируется и обрабатывается только этим элементом - это прямое событие.

Пример пузырькового события:
```csharp
<Button x:Name="helpButton" Click="HelpButton_OnClick" Height="30">
    <StackPanel Orientation="Horizontal">
        <Label Height="30">Кнопка</Label>
        <Ellipse Name="ellipse" Fill="Green" Height="25" Width="25"/>
    </StackPanel>
</Button>
private void HelpButton_OnClick(object sender, RoutedEventArgs e)
{
    MessageBox.Show("Кнопка нажата");
}
```
Пример прекращения распространения пузырькового события:
```csharp
<Button x:Name="helpButton" Click="HelpButton_OnClick" Height="30">
    <StackPanel Orientation="Horizontal">
        <Label Height="30">Кнопка</Label>
        <Ellipse Name="ellipse" Fill="Green" Height="25" Width="25" MouseDown="Ellipse_OnMouseDown"/>
    </StackPanel>
</Button>
private void HelpButton_OnClick(object sender, RoutedEventArgs e)
{
    MessageBox.Show("Кнопка нажата");
}
private void Ellipse_OnMouseDown(object sender, MouseButtonEventArgs e)
{
    this.Title += " Эллипс!";
    e.Handled = true; //остановить распространение
}
```
Туннельное событие (имена начинаются с Preview) спускаются с самого верхнего элемента до внутренних областей дерева объектов. Перед возникновением пузырькового события MouseDown сначала инициируется туннельное событие PreviewMouseDown.
```csharp
<Button x:Name="helpButton" Click="HelpButton_OnClick" Height="30">
    <StackPanel Orientation="Horizontal">
        <Label Height="30">Кнопка</Label>
        <Ellipse Name="ellipse" Cursor="Hand" Fill="Green" Height="25" Width="25" MouseDown="Ellipse_OnMouseDown" PreviewMouseDown="Ellipse_OnPreviewMouseDown"/>
    </StackPanel>
</Button>
private string mouseActivity = string.Empty;
private void AddEventInfo(object sender, RoutedEventArgs e)
{
    mouseActivity += $"{sender} посылает {e.RoutedEvent.RoutingStrategy} событие с названием {e.RoutedEvent.Name}\n";
}
private void HelpButton_OnClick(object sender, RoutedEventArgs e)
{
    AddEventInfo(sender,e);
    MessageBox.Show(mouseActivity,"Кнопка нажата");
    mouseActivity = "";
}
private void Ellipse_OnMouseDown(object sender, MouseButtonEventArgs e)
{
    AddEventInfo(sender,e);
}
private void Ellipse_OnPreviewMouseDown(object sender, MouseButtonEventArgs e)
{
    AddEventInfo(sender,e);
}
```










