# Домены приложений (процессы, контексты)

Для каждого загруженного в паять файла *.exe ОС создает изолированный процесс, импеющий свой униакльный идентификатор процесса PID. Данные в одном процессе не доступны напрямую другим процессам, если не применяется API.

Каждый процесс содержит начальный поток, начинающийся с метода Main(). API ОС и .NET позволяют главному потоку возможность порождения дополнительных вторичных (рабочих) потоков. После запуска вторичного потока главный поток продолжает реагировать на пользовательский ввод.

# Взаимодействие с процессами

Пространство имен для взаимодействия с процессами System.Diagnostics, некоторые типы:
```csharp
Process                 Доступ к локальным и удаленным процессам, запуск и стоп процессов
ProcessModule           Дает модуль, загруженный в опредленный процесс
ProcessModuleCollection Дает коллекцию ProcessModule
ProcessStartInfo        Набор значений, применяемых для запуска Process.Start()
ProcessThread           Дает поток внутри заданного процесса
ProcessThreadCollection Дает коллекцию ProcessThread
```

Свойства некоторые класса Process:
```csharp
ExitTime                Метка времени, ассоциированная с процессом, когда завершен
Handle                  Дескриптор, который назначен процессу операционной системой
Id                      Идентификатор ID связанного процесса
MachineName             Получение имени компьютера, на котором исполняется компьютер
MainWindowTitle         Загловок главного окна
Modules                 Доступ к строго типизированной коллекции модулей dll и exe, которые были загружены внутри этого процесса
ProcessName             Имя процесса
Responding              Значение, указывающее, реагирует ли пользовательский интерфейс процесса на пользовательский ввод
StartTime               Значение времени, когда был запущен процесс
Threads                 Дает набор потоков, выполняемых в связанном процесса - коллекция ProcessThread
```

Некоторые методы класса Process
```csharp
CloseMainWindow()       Закрывает процесс, содержащий пользовательский интерфейс
GetCurrentProcess()     Стстический метод Возвращает новый объект Process, представляющий процесс активный в данный момент
GetProcesses()          Статический метод Дает массив объектов - процессов Process, которые выполняются на заданной машине
Kill()                  Останавливает связанный процесс
Start()                 Метод запускает процесс
```

Некоторые методы класса ProcessThread
```csharp
CurrentPriority         Текущий приоритет процесса
Id                      Идентификатор потока
IdealProcessor          Устанавливает предпочитаемый процессор для этого потока
PriorityLevel           Уровень приоритета
ProcessorAffinity       Устанавливает процессоры, на которых может выполняться этот поток
StartAddress            Дает адрес в памяти функции, вызванной операционной системой, которая запустила этот поток
StartTime               Время, когда ОС запустила поток
ThreadState             Текущее состояние потока
TotalProcessorTime      Общее время, потраченное этим потоком на использование процессора
WaitReason              Причина, по которой поток находится в состоянии ожидания
```

Пример просмотра выполняющихся процессов:
```csharp
public static void Main()
{
    var procs = from proc in Process.GetProcesses(".")
        orderby proc.Id
        select proc; //все процессы на локальной машине
    foreach (var p in procs)
    {
        Console.WriteLine($"-> PID: {p.Id}\tName: {p.ProcessName}");
    }
    Console.ReadLine();
}
```

Пример получения определенного процесса и просмотр его потоков:
```csharp
public static void Main()
{
    Process myProc = null;
    try
    {
        myProc = Process.GetProcessById(888);
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.Message);
        return;
    }
    Console.WriteLine($"Название процесса: {myProc.ProcessName}");
    var myThreads = myProc.Threads;
    foreach (ProcessThread t in myThreads)
    {
        Console.WriteLine($"-> Поток Id: {t.Id}\tЗапущен:{t.StartTime}\tПриоритет:{t.PriorityLevel}");
    }
    Console.ReadLine();
}
```
Пример получения загруженных модулей процесса отдельного потока:
```csharp
public static void Main()
{
    Process myProc = null;
    try
    {
        myProc = Process.GetProcessById(10024);
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.Message);
        return;
    }
    Console.WriteLine($"Название процесса: {myProc.ProcessName}");
    var myModules = myProc.Modules;
    foreach (ProcessModule m in myModules)
    {
        Console.WriteLine($"-> Название подуля:{m.ModuleName}");
    }
    Console.ReadLine();
}
```
Пример запуска и остановки процесса:
```csharp
public static void Main()
{
    Process myProc = null;
    try
    {
        ProcessStartInfo startInfo = new ProcessStartInfo("firefox.exe", "www.yandex.ru");
        startInfo.WindowStyle = ProcessWindowStyle.Maximized; //развернуть окно
        myProc = Process.Start(startInfo);
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.Message);
        return;
    }
    Console.WriteLine($"Нажмите клавишу для закрытия процесса '{myProc.ProcessName}'");
    Console.ReadKey();
    try
    {
        myProc.Kill();
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.Message);
    }
    Console.ReadLine();
}
```

## Домены приложений

Исполняемый файл .NET попадает в отдельный раздел логического процесса, называемый доменом приложения. Каждый процесс может содержать несколько доменов.

- домены приложений обеспечивают нейтральное отношение к ОС платформы .NET.

- домены гораздо менее затратны по вычислительными ресурсами по сравнению с процессами.

- домены более изолированны при размещении загруженных приложений.

Один процесс ОС будет обслуживать как минимум один стандартный домен приложения, затем среда CLR может создавать дополнительные домены приложений.

Некоторые методы класса AppDomain:
```csharp
CreateDomain()          Создать новый домен приложения
CreateInstance()        Создать экземпляр типа из внешней сборки
ExecuteAssembly()       Выполняет сборку *.exe внутри домена
GetAssemblies()         Получает набор сборок .NET, которые были загружены
GetCurrentThreadId()    Вертает идентификатор активного потока в текущем домене приложения
Load()                  Статический метод динамическая загрузка сборки в текущий домен приложения
Unload()                Статический метод выгрузки домена из приложения
```
Некоторые свойства класса AppDomain:
```csharp
BaseDirectory           Свойство путь к каталогу, где зондируются сборки
CurrentDomain           Домент приложения
FriendlyName            Дружественное имя текущего домена
MonitoringIsEnabled     Включен ли мониторинг ЦП и памяти, потребляемых доменами приложений
SetupInformation        Детали конфигурации для указанного домена приложения
```
Некоторые события класса AppDomain:
```csharp
AssemblyLoad            Когда сборка загружается в память
AssemblyResolve         Когда распознователь сборок не может найти местоположение обязательной сборки
DomainUnload            Перед началом выгрузки домена приложения из процесса
FirstCancelException    Уведомление о том, что было сгенерировано исключение, перед тем, как будет поиск оператора catch
ProcessExit             Когда родительский процесс завершается
UnhantledException      Когда исключение не было перехвачено обработчиком
```

Примеры работы с доменом приложения:
```csharp
public static void Main()
{
    AppDomain defAD = AppDomain.CurrentDomain;
    Console.WriteLine($"Дружественное имя: {defAD.FriendlyName}");
    Console.WriteLine($"Id домена в процессе: {defAD.Id}");
    Console.WriteLine($"Этот домен стандартный: {defAD.IsDefaultAppDomain()}");
    Console.WriteLine($"Базовый каталог: {defAD.BaseDirectory}");
    Console.ReadLine();
}
```
Загруженные сборки:
```csharp
public static void Main()
{
    AppDomain defAD = AppDomain.CurrentDomain;
    Assembly[] loadedAss = defAD.GetAssemblies();
    Console.WriteLine($"Сборки заргужены в стандартный домен {defAD.FriendlyName}");
    foreach (var a in loadedAss)
    {
        Console.WriteLine($"-> Имя: {a.GetName().Name} Версия: {a.GetName().Version}");
    }
    Console.ReadLine();
}
```
c LINQ:
```csharp
public static void Main()
{
    AppDomain defAD = AppDomain.CurrentDomain;
    var loadedAss = defAD.GetAssemblies().OrderBy(a => a.GetName().Name).Select(a=>a);
    Console.WriteLine($"Сборки заргужены в стандартный домен {defAD.FriendlyName}");
    foreach (var a in loadedAss)
    {
        Console.WriteLine($"-> Имя: {a.GetName().Name} Версия: {a.GetName().Version}");
    }
    Console.ReadLine();
}
```
Уведомление о загрузке сборок:
```csharp
public static void Main()
{
    AppDomain defAD = AppDomain.CurrentDomain;
    defAD.AssemblyLoad += (sender, args) =>
    {
        Console.WriteLine($"{args.LoadedAssembly.GetName().Name} загружена сборка.");
    };
    Console.ReadLine();
}
```
Создание новых доменов:
```csharp
public static void Main()
{
    AppDomain defAD = AppDomain.CurrentDomain;
    ListAssemblies(defAD);
    AppDomain newAD = AppDomain.CreateDomain("SecondAppDomain");
    ListAssemblies(newAD);
    Console.ReadLine();
}
private static void ListAssemblies(AppDomain defAD)
{
    var loadedAss = defAD
        .GetAssemblies()
        .OrderBy(l => l.GetName().Name)
        .Select(l => l);
    Console.WriteLine($"Загруженные сборки в {defAD.FriendlyName}:");
    foreach (var l in loadedAss)
    {
        Console.WriteLine($"Имя: {l.GetName().Name} Версия: {l.GetName().Version}");
    }
}
```
Загрузка сборки в новый домен:
```csharp
public static void Main()
{
    AppDomain newAD = AppDomain.CreateDomain("SecondAppDomain");
    try
    {
        newAD.Load("ClassLibrary1");
    }
    catch (Exception e)
    {
        Console.WriteLine(e.Message);
    }
    ListAssemblies(newAD);
    Console.ReadLine();
}
private static void ListAssemblies(AppDomain defAD)
{
...
}
```
Выгрузка сборки из нового домена:
```csharp
public static void Main()
{
    AppDomain newAD = AppDomain.CreateDomain("SecondAppDomain");
    newAD.DomainUnload += (sender, args) =>
    {
        Console.WriteLine("Второй домен приложения выгружен!");
    };
    try
    {
        newAD.Load("ClassLibrary1");
    }
    catch (Exception e)
    {
        Console.WriteLine(e.Message);
    }
    ListAssemblies(newAD);
    AppDomain.Unload(newAD);
    Console.ReadLine();
}
private static void ListAssemblies(AppDomain defAD)
{
    var loadedAss = defAD
        .GetAssemblies()
        .OrderBy(l => l.GetName().Name)
        .Select(l => l);
    Console.WriteLine($"Загруженные сборки в {defAD.FriendlyName}:");
    foreach (var l in loadedAss)
    {
        Console.WriteLine($"Имя: {l.GetName().Name} Версия: {l.GetName().Version}");
    }
}
```

## Контексты

Каждый домен приложения может дополнительно разделен на многочисленные контектные границы.

Каждый процесс определяет стандартный домен приложения, а каждый домен приложения уже имеет стандартный контекст, он называется контекст 0. Подавляющее число объектов загружаются в контекст 0. И только если среда CLR понимает, что созданный объект предъявляет спецтребования, то тогда она создает внутри домена новую контексную границу.

Пример контекстно-связанного объекта:
```csharp
[Synchronization] //требует загрузки в синхронизированный контекст
class MyClass : ContextBoundObject
{
    public MyClass()
    {
        Context ctx = Thread.CurrentContext;
        Console.WriteLine($"{this.ToString()} в контексте {ctx.ContextID}");
        foreach (var c in ctx.ContextProperties)
        {
            Console.WriteLine($"-> Свойство: {c.Name}");
        }
    }
}
class Program
{
    public static void Main()
    {
        MyClass my = new MyClass();
        Console.ReadLine();
    }
}
```



