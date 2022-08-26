# Основы

Программисты .NET имеют единственный симметричный API-интерфейс для всех распространенных потребностей при использовании WPF для разработки графических настрольных приложений.

## Инфраструктура

Инфраструктура WPF определеяет способ отделения внешнего вида и поведения приложения с графическим пользовательским интерфейсом от программной логики. Пользовательский интерйейс пишется на языке XAML через разметку XML - "настольную разметку", с помощью инструментов Microsoft Visual Studio или Microsoft Expression Blend, затем может быть подключена к связанном файлу кода. Допустимо изменять внешний вид элемента управления только с помощью разметки. Модель WPF отличается от других тем, что визуализация графических данных происходит через API-интерфейс DirectX, а GDI не используется. Приложения WPF автоматом получают аппараткую и программную оптимизацию, могут задействовать графические службы без сложностей, присущих программированию напрямую через API DirectX.

WPF - API-интерфейс, предназначенный для построения настольных приложений, который интегрирует разнообразные настольные API-интерфейсы в единую объектную модель и обеспечивает четкое разделение обязанностей через XAML.

Основные функциональные возможности WPF:

- множество диспетчеров компоновки для гибкого контроля над размещением и изменением содержимого

- расширенный механизм привязок данных для связывания содержимого с элементами пользовательского интерфейса

- встроенный механизм стилей, позволяющий определять темы приложения

- векторная графика для автоматического изменения размеров содержимого с целью соответтсвия размерам и разрешению экрана

- поддержка двухмерной и трехмерной графики, анимации, аудио и видео

- типографичкский API-интерфейс, поддерживающий документы XML, фиксированные документы, нефиксированные документы и аннотации в документах

- взаимодействие с унаследованными моделями графических пользовательских интерфейсов

## Введение

Инфраструктура WPF - коллекция типов, втроенных в сборки .NET.

Класс System.Windows.Application - глобальный экземпляр выполняющегося приложения WPF с методом Run() для запуска приложения и набором событий для обработки взаимодействия с приложением.

Методы этого класса:

- Current - Статическое свойство получение доступа к работающему объекта Application из любого места кода

- MainWindow - Свойство главного окна приложения

- Properities - Свойство данных, доступных через все аспекты прилжения WPF

- StartupUri - Свойство указывающее окно или страница для автоматического открытия при запуске приложения

- Windows - Дает объект типа WindowCollection, дающее доступ ко всем окнам, созданным в потоке, создавшем объект Application.

Класс System.Windows.Controls.ContentControl является родительским для класа Window, срабжающий все производные класса способностью размещать в себе одиночный фрагмент содержимого через свойство Content.

Пример:

```csharp
<Button Height="40" Width="80" Content="OK"></Button>
```

Класс System.Windows.Controls.Control является базовым для всех элементов управления WPF, снабжая их основной функциональностью пользовательского интерфейса.

Класс System.Windows.FrameworkElement дает несколько членов применяемых повсюду для раскадровки, привязки данных, именования элементов, получения ресурсов и установки общих измерений производноо типа.

Класс System.Windows.UIElement обеспечивает наибольший объем функциональности, дает производному типу многочисленные события, чтоб мог получать фокус и обрабатывать входные запросы.

Класс System.Windows.Media.Visual дает поддержку визуализации в WPF.

Класс System.Windows.DependencyObject дает поддержку разновидности свойств - свойств зависимости - в WPF.

Класс System.Windows.Threading.DispatcherObject дает поддержку очередни событий в WPF, для организации параллелизма и многопоточности.

## Разметка

Корневой элемент разметки почти всегда ссылается на два заранее определенных пространства имен:

```csharp
xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
```

Первое пространство имен XML отображает множество связанных с WPF пространств для использования текущим файлом *.xaml - Windows, Controls, Data, Ink, Media, Navigation.

Второе пространство имен XML служит для добавления специфичных для XAML "ключевых слов".

Помимо ключевых слов x:Name, x:Class, x:Code пространство имен http://schemas.microsoft.com/winfx/2006/xaml предоставляет несколько дополнительных ключевых слов.

Для добавления новых элементов из других пространств имен используется синтаксис такой в корневом элементе XML:

```csharp
xmlns:myns="clr-namespace:MyControls;assembly=MyControls" //во внешней сборке
xmlns:myns="clr-namespace:MyNewNamespace" //пространство имен в этой сборке
//элемент:
<myns:MyElement/>
```

По умолчанию все типы C#/XAML являются открытыми, а члены - внутренними. Это можно изменять используя ключевые слова ClassModifier & FieldModifier, пример:

```csharp
<Grid x:ClassModifier="internal">
    <Button Name="myButton" x:FieldModifier="public" Content="OK"></Button>
</Grid>
```

Элементы XAML отображаются на типы классов или структур внутри заданного пространства имен .NET, а атрибуты в открывающем дескрипторе элемента отображаются на свойства или события этого типа. Когда нужно присвоить сложный объект в качестве значения свойства - нужно использовать синтаксис "свойство-элемент".

Пример присвоения сложного свойства через синтаксис "свойство-элемент":
```csharp
<Button Height="40" Content="OK" FontSize="20" Foreground="Yellow">
    <Button.Background>
        <LinearGradientBrush StartPoint="1,0" EndPoint="1,1">
            <GradientStop Color="LightGreen" Offset="0"/>
            <GradientStop Color="DarkGreen" Offset="1"/>
        </LinearGradientBrush>
    </Button.Background>
    <Button.Width>
        100
    </Button.Width>
</Button>
```

В XAML поддерживается специальный синтаксис для установки значения присоединяемого свойства, позволяющее дочернему элементу устанавливать значение свойства, которое на самом деле определено в родительском элементе. 

Пример применения такого свойства зависимости:

```csharp
<Grid>
    <Button Grid.Column="0" Grid.Row="0" Height="40" Width="100" FontSize="20">
        OK
    </Button>
</Grid>
```

Существует еще один способ установки значения атрибута XAML - расширенная разметка. Она позволяет получить значение для свойства из выделенного внешнего класса. Элементы расширенной разметки помещаются между фигурными скобками.

Пример расширенной разметки:

```csharp
<Page xmlns:corlib="clr-namespace:System;assembly=mscorlib"
    ...>
    <StackPanel>
        <!-- Получение значения статического члена класса -->
        <Label Content="{x:Static corlib:Environment.OSVersion}"></Label>
        <!-- Получение типа, операция typeof языка C# -->
        <Label Content="{x:Type Button}"></Label>
        <Label Content="{x:Type corlib:Int32}"></Label>
        <!-- Наполнение элемента массивом строк -->
        <ListBox Margin="10">
            <ListBox.ItemsSource>
                <x:Array Type="corlib:String">
                    <corlib:String>Первая строка</corlib:String>
                    <corlib:String>Вторая</corlib:String>
                    <corlib:String>Третья строка</corlib:String>
                </x:Array>
            </ListBox.ItemsSource>
        </ListBox>
    </StackPanel>
</Page>
```

C помощью XAML можно описывать любой тип из любой сборки при условии, что он является неабстрактным и содержит стандартный конструктор.

## Строительство приложения

Строительство типичного WPF приложения. После создания проекта WPF по умолчанию утанавливаются ссылки на все сборки WPF (PresentationCore.dll, PresentationFramework.dll, System.Xaml.dll и WindowsBase.dll) в XAML файле и файле кода C#.

В файле App.xaml посредством разметки определен класс приложения. Там определены свойства приложения, такие как StartupUri (окно, подлежащее загружке при запуске), ресурсы уровня приложения и специфические обработчики для событий приложения вроде Startup и Exit. Пример разметки и добавленных обработчиков событий запуска приложения и закрытия приложения:

Файл App.xaml & App.xaml.cs:

```csharp
<Application x:Class="WpfApp1.App"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:local="clr-namespace:WpfApp1"
             StartupUri="MainWindow.xaml"
             Startup="App_OnStartup" Exit="App_OnExit">
    <Application.Resources>
    </Application.Resources>
</Application>
public partial class App : Application
{
    private void App_OnStartup(object sender, StartupEventArgs e)
    {
    }
    private void App_OnExit(object sender, ExitEventArgs e)
    {
    }
}
```

Утилита msbuild.exe при построении приложения для каждого файла XAML создает в проекте три файла: *.g.cs (g - автогенерируемый), *.g.i.cs (i - IntelliSense) и *.baml (BAML - двоичный язык разметки приложений). Эти файлы сохраняются в папке \obj\Debug.

А в файле BAML - компактное двоичное представление исходных данных XAML. Он встраивается в виде ресурса в скомпилированную сборку, он встраивается в конечное приложение и при каждом выполенеии приложения он извлекается из контейнера ресурсов и применяется для настройки внешнего вида и поведения всех окон и элементов управления.

В файле App.g.cs содержится автоматически сгенерированный метод Main(), инициализирующий и запускающий приложение. Метод InitializeComponent() конфигурирует свойства приложения, запускаемое окно и обрботчики событий.

В классе Application есть свойство Properties, позволяющее определить коллекцию пар "имя/значение" через индексатор типа System.Object. В эту коллекцию можно сохранять все что угодно с целью последующего извлечения по дружественному имени:

```csharp
Application.Current.Properties["Test"] = true;
if ((bool) Application.Current.Properties["Test"])
    MessageBox.Show("Есть!");
```

Инфраструктура WPF предлагает события для выяснения, действительно ли пользователь намерен закрыть окно. Одно из них - Cloaing в сочетании с делегатом CancelEventHandler, а он предоставляет свойство Cancel, установка оного в true предотвращает закрытие окна. А другое событие - Closed - точка, где окно полностью и безвозвратно готово к закрытию.

Пример применения запроса на подтверждение закрытия окна с данными событиями:

```csharp
public MainWindow()
{
    InitializeComponent();
    this.Closing += MainWindow_Closing;
    this.Closed += MainWindow_Closed;
}
private void MainWindow_Closing(object sender, System.ComponentModel.CancelEventArgs e)
{
    MessageBoxResult result = MessageBox.Show("Действительно закрыть окно?", "Приложение",
        MessageBoxButton.YesNo, MessageBoxImage.Question);
    if (result == MessageBoxResult.No)
        e.Cancel = true; //отмена закрытия окна
}
private void MainWindow_Closed(object sender, EventArgs e)
{
    MessageBox.Show("Пока!");
}
```

Инфраструктура WPF дает несколько событий, помогающих взаимодействоввать с мышью. Определяются связанные с мышью события - MouseMove, MouseUp, MouseDown, MouseEnter, MouseLeave и др. Метод GetPosition() позволяет получать значение (x, y) относительного элемента пользовательского интерфейса в окне.

Пример:

```csharp
public MainWindow()
{
    InitializeComponent();
    this.MouseMove += MainWindow_MouseMove;
}
private void MainWindow_MouseMove(object sender, MouseEventArgs e)
{
    this.Title = e.GetPosition(this).ToString();
}
```

Очень проста обработка клавиатурного ввода окна, на котором находится фокус. Это события - KeyUp, KeyDown, работающие с делегатом System.Windows.Input.KeyEventHandler.

Пример:

```csharp
public MainWindow()
{
    InitializeComponent();
    this.KeyDown += MainWindow_KeyDown;
}
private void MainWindow_KeyDown(object sender, KeyEventArgs e)
{
    this.Title = e.Key.ToString();
}
```

