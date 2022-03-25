# Библиотека полезностей

## Класс математических операций
```csharp
Math.Pow(a,b); //возаращает а в степери b
```

## Класс получения расположения исполняемой сборки приложения
```csharp
string initDir = Path.GetDirectoryName(Assembly.GetExecutingAssembly().Location);
Console.WriteLine(initDir);
```

## Получение сведений о текущей среде выполнения и платформе
```csharp
Console.WriteLine(Environment.CommandLine);
Console.WriteLine(Environment.CurrentDirectory);
Console.WriteLine(Environment.CurrentManagedThreadId);
Console.WriteLine(Environment.ExitCode);
Console.WriteLine(Environment.HasShutdownStarted);
Console.WriteLine(Environment.Is64BitOperatingSystem);
Console.WriteLine(Environment.Is64BitProcess);
Console.WriteLine(Environment.MachineName);
Console.WriteLine(Environment.NewLine);
Console.WriteLine(Environment.OSVersion);
Console.WriteLine(Environment.ProcessorCount);
Console.WriteLine(Environment.StackTrace);
Console.WriteLine(Environment.SystemDirectory);
Console.WriteLine(Environment.SystemPageSize);
Console.WriteLine(Environment.TickCount);
Console.WriteLine(Environment.UserDomainName);
Console.WriteLine("--");
Console.WriteLine(Environment.UserInteractive);
Console.WriteLine(Environment.UserName);
Console.WriteLine(Environment.Version);
Console.WriteLine(Environment.WorkingSet);
Console.WriteLine("--получение аргументов строки запуска приложения");
foreach (var item in Environment.GetCommandLineArgs())
{
    Console.WriteLine(item);
}
```


## Класс работы с массивом 
Array
```csharp
Имя метода      Тип             Описание
Clear           Статический     Присваивает элементам массива значение 0,
                                false или null в зависимости от типа элементов
GetLength       Нестатический   Получает целое число, представляющее 
                                количество элементов в заданном измерении              
Sort            Статический,    Сортирует элементы во всем одномерном массиве Array 
                                перегруженный                           
```

## Класс случайных значений
```csharp
Random rnd = new Random();
int variable = rnd.Next(0, 100);
```

## Замер скорости выполения кода:
```csharp
Stopwatch stopwatch = new Stopwatch();
stopwatch.Start();
/////////код для замера
stopwatch.Stop();
var elapsedTime = stopwatch.Elapsed; //результат работы
```
Получение прошедшего времени:
```csharp
DateTime start=DateTime.Now;
System.Threading.Thread.Sleep(20);// делаем паузу
DateTime finish=DateTime.Now;
Console.WriteLine(finish-start);
```
Еще пример получения прошедшего времени:
```csharp
var start = DateTime.Now;
/////////код для замера
var res = (DateTime.Now - start).TotalMilliseconds;
Console.WriteLine("время - {0} мс",res);
```

## Точный замер скорости выполнения кода и количества сборок мусора:
```csharp
internal sealed class OperationTimer : IDisposable
{
    private int _startTime;
    private string _text;
    private string _collectionCount;
    public OperationTimer(string text)
    {        
        GC.Collect();
        GC.WaitForPendingFinalizers();
        GC.Collect();
        _text = text;
        _collectionCount = GC.CollectionCount(0);
        _startTime = Stopwatch.StartNew();
    }
    public void Dispose()
    {
        Console.WriteLine("{0} (GC={1,3}) - {2}", _startTime.Elapsed, GC.CollectionCount(0) - _collectionCount, _text);
    }
}
```
Пример использования:
```csharp
using (new OperationTimer("List<int>"))
{
    List<int> list = new List<int>();
    for (var i = 0; i < 100000; i++)
    {
        list.Add(i);
        int x = list[i];
    }
    list = null;
}
```

## Цикличная задержка программы в асинхронном режиме
```csharp
public static void Main()
{
    Run();
    Console.ReadLine();
    async void Run()
    {
        while (true)
        {
            Console.WriteLine("+");
            await Task.Delay(1000);
        }
    }
}
```

## Конфигурационный XML файл сборки.
Допустимо использовать набор типов из пространства имен System.Configuration для специальных настроек приложения.

Файл "App.config":
```csharp
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
...
    <appSettings>
      <add key="Str1" value="Green"/>
      <add key="Number1" value="8"/>
    </appSettings>
```
Использование:
```csharp
using System.Configuration;
namespace ConsoleApp1
{
    class Program
    {
        public static void Main()
        {
            AppSettingsReader ar = new AppSettingsReader();
            int numb1 = (int)ar.GetValue("Number1", typeof(int));
            string str1 = (string) ar.GetValue("Str1", typeof(string));
            Console.WriteLine($"Проверка: {str1} {numb1}");
```

Пример получения конфигурации соединения:

Файл "App.config":
```csharp
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
...
  <connectionStrings>
    <add name="DefaultConnection" 
         connectionString="Data Source=(localdb)\MSSQLLocalDB;
                          Initial Catalog=demo06022020;
                          Integrated Security=True;"/>
```
Использование:
```csharp
var connectionString = ConfigurationManager.ConnectionStrings["DefaultConnection"].ConnectionString;
```

Пример чтения конфигурации в переменную-перечисление:

Файл "App.config":
```csharp
...
    <appSettings>
      <add key="MySetting" value="Test1"/>
    </appSettings>
```
Использование:
```csharp
enum MyEnum
{
    Test1,
    Test2,
}
private MyEnum _setting;
public MainWindow()
{
    InitializeComponent();
}
private void MainWindow_OnLoaded(object sender, RoutedEventArgs e)
{
    string str = ConfigurationManager.AppSettings["MySetting"];
    if (Enum.IsDefined(typeof(MyEnum), str))
    {
        _setting = (MyEnum)Enum.Parse(typeof(MyEnum), str);
    }
    MessageBox.Show($"Настрока: {_setting.ToString()}");
}
```


