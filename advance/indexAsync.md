# Асинхронное, параллельное, многопоточние программирование, TPL, Timer, PLINQ

Заданный домен приложения может иметь многочисленные потоки, выполняющиеся в каждый конктетный момент времени. Потоки могут перемещаться между границами доменов приложений, но каждый может выполняться только внутри одного домена приложения.

Одиночный поток может быть перенесен в определенный контекст и перемещаться внутри нового контекста.

За перемещение потоков между доменами приложений и контекстами отвечает среда CLR.

Многопоточные программы изменчивы, множество потоков могут оперировать разделяемыми ресурсами одновременно. Чтобы защиить ресурсы от повреждения, нужно применять потоковые примитивы - блокировки, мониторы, атрибут [Syncranization], либо поддержка языковых ключевох слов для управления доступом между потоками.

Пример получения информации о потоке:
```csharp
public static void Main()
{
    Thread currT = Thread.CurrentThread;
    AppDomain appDomain = Thread.GetDomain();
    Console.WriteLine($"Домен приложения: {appDomain.FriendlyName}");
    Context ctx = Thread.CurrentContext;
    Console.WriteLine($"Контекст: {ctx.ContextID}");
    Console.ReadLine();
}
```

## Асинхронное программирование async/await

В версии .NET 4.5 появились async и await для упрощения процесса написания асинхронного кода. Компилятор будет самостоятельно генерировать большой объем кода, связанного с потоками, из пространств System.Threading и System.Theading.Tasks.

Ключевое слово async помечает метод, лямбда-выражение или анонимный метод должен вызыватсься в асинхронной манере автоматически. При вызове async ключевое слово await будет автоматически приостанавливать текущий поток, пока задача не завершится, давая возможность вызывающему потоку продолжить свою работу.

Асинхронный метод должен быть помечен async и меть в своем коде хотя бы один await, иначе он будет синхронным.

Пример использования асинхронных методов с одним входным параметром:
```csharp
static void MyWork()
{
    Thread.Sleep(3_000);
    Console.WriteLine("void Работа");
}
static void MyDetailWork(int x)
{
    Thread.Sleep(3_000);
    Console.WriteLine($"void Подробная работа #{x}");
}
static async void MyVoidAsync() //без возврата значения
{
    await Task.Run(() =>
    {
        Thread.Sleep(5_000);
        Console.WriteLine("void async Работа");
    });
}
static async Task MyWorkAsync(int x)
{
    await Task.Run(MyWork); //без параметра
    await Task.Run(() => MyDetailWork(x)); //с параметром
    Thread.Sleep(2_000);
    Console.WriteLine($"async Task Работа #{x}");
}
static async Task Main() //вызов в неблокирующей манере
{
    MyWorkAsync(1); //вызов асинхронного метода
    MyVoidAsync(); //еще вызов асинх метода
    Console.WriteLine("Работа #1 запущена асинхронно");
    await Task.Run(() =>
    {
        Thread.Sleep(5_000);
        Console.WriteLine("Лямбда работа");
    }); //асинхронная работа лямбда функция с ожиданием, хотя можно без, если удалить await
    Console.WriteLine("Введите что нибудь:");
    string s = Console.ReadLine();
    Console.WriteLine($"Вы ввели: {s}");
    Console.WriteLine("Старт и ожидание завершения работы #2");
    await MyWorkAsync(2); //ожидание завершения асинх метода
    Console.WriteLine("Работа #2 завершена");
    Console.WriteLine("Нажмите любую кнопку ...");
    Console.ReadKey();
}
```
Пример использования асинхронных функций с входным и возвращаемым параметром:
```csharp
static int MyFunc(int x)
{
    Thread.Sleep(3_000);
    Console.WriteLine($"int Вычисление : {x}");
    return x + 1;
}
static async Task<int> MyFuncAsync(int x)
{
    int y = await Task.Run(() => MyFunc(x));
    Thread.Sleep(2_000);
    Console.WriteLine($"int async Вычисление : {y}");
    return y + 1;
}
static async Task<int> MyFuncSumAsync() //сумма асинхронная вычислений
{
    Task<int> t1 = Task<int>.Run(() => MyFunc(1));
    Task<int> t2 = Task<int>.Run(() => MyFunc(2));
    Task<int> t3 = Task<int>.Run(() => MyFunc(3));
    var res = await Task<int>.WhenAll(new[] {t1, t2, t3}); //выполнение всех задач
    return res.Sum();
}
static async Task Main() //вызов в неблокирующей манере
{
    int x = await Task<int>.Run(() => 1); //асинхронная работа лямбда функция с ожиданием
    Console.WriteLine("Введите что нибудь:");
    string s = Console.ReadLine();
    Console.WriteLine($"Вы ввели: {s}");
    Console.WriteLine("Старт и ожидание завершения вычислений");
    int y = await MyFuncAsync(1); //ожидание завершения асинх метода
    Console.WriteLine($"Вычисление завершено y = {y}");
    y = await MyFuncSumAsync();
    Console.WriteLine($"Результат вычисления асинхронного: {y}");
    Console.WriteLine("Нажмите любую кнопку ...");
    Console.ReadKey();
}
```
Пример вызова обычных функций асинхронно:
```csharp
static void MyWork()
{
    Thread.Sleep(6_000);
    Console.WriteLine("void Работа");
}
static int MyFunc(int x)
{
    Thread.Sleep(3_000);
    Console.WriteLine($"int Вычисление : {x}");
    return x + 1;
}
static async Task Main() //вызов в неблокирующей манере
{
    Task.Run(MyWork); //запуск асинхронно
    Console.WriteLine("Работа запущена асинхронно");
    Console.WriteLine("Старт и ожидание завершения вычислений");
    int y = await Task<int>.Run(() => MyFunc(1)); //ожидание завершения асинх вычислений
    Console.WriteLine($"Вычисление завершено y = {y}");
    Console.WriteLine("Нажмите любую кнопку ...");
    Console.ReadKey();
}
```

## Многопоточное программирование

При формировании делегата C# автоматически генерирует класс, в котором определены два метода BeginInvoke() и EndInvoke():
```csharp
public delegate int MySample(int x, int y);
//формируется вот такой класс:
public sealed class MySample : System.MulticastDelegate
{
    public MySample(object target, uint functionAddress);
    public int Invoke(int x, int y);
    public IAsyncResult BeginInvoke(int x, int y, AsyncCallback cb, object state);
    public int EndInvoke(IAsyncResult result);
}
```
Пример асинхронного вызова метода:
```csharp
public static void Main()
{
    Console.WriteLine($"Main() запущен в потоке: {Thread.CurrentThread.ManagedThreadId}");
    MySample mySmp = new MySample(Add);
    IAsyncResult ar = mySmp.BeginInvoke(10, 10, null, null);
    Console.WriteLine("Выполение другой работы в Main()");
    int answer = mySmp.EndInvoke(ar);
    Console.WriteLine($"Результат: 10 + 10 = {answer}");
    Console.ReadLine();
    int Add(int x, int y)
    {
        Thread.Sleep(5000);
        return x + y;
    }
}
```
Пример асинхронного вызова метода в наименее блокирующей манере:
```csharp
public static void Main()
{
    Console.WriteLine($"Main() запущен в потоке: {Thread.CurrentThread.ManagedThreadId}");
    MySample mySmp = new MySample(Add);
    IAsyncResult ar = mySmp.BeginInvoke(10, 10, null, null);
    while (!ar.IsCompleted)
    {
        Console.WriteLine("Ожидание выполнения в Main()");
        Thread.Sleep(1000);
    }
    int answer = mySmp.EndInvoke(ar);
    Console.WriteLine($"Результат: 10 + 10 = {answer}");
    Console.ReadLine();
    int Add(int x, int y)
    {
        Thread.Sleep(5000);
        return x + y;
    }
}
```
Пример использования делегата AsyncCallback:
```csharp
public delegate int MySample(int x, int y);
class Program
{
    private static bool isDone = false;
    public static void Main()
    {
        Console.WriteLine($"Main() запущен в потоке: {Thread.CurrentThread.ManagedThreadId}");
        MySample mySmp = new MySample(Add);
        //IAsyncResult asyncRes = mySmp.BeginInvoke(10, 10, null, null); //без передачи делегата
        //IAsyncResult asyncRes = mySmp.BeginInvoke(10, 10, AddComplete, null); //передача делегата обратного вызова
        IAsyncResult asyncRes = mySmp.BeginInvoke(10, 10, AddComplete, "Привет шахтерам!"); //передача делегата обратного вызова и данных
        while (!isDone)
        {
            Console.WriteLine("Ожидание выполнения в Main()");
            Thread.Sleep(1000);
        }
        Console.ReadLine();
        int Add(int x, int y)
        {
            Console.WriteLine($"Add() запущен в потоке {Thread.CurrentThread.ManagedThreadId}");
            Thread.Sleep(5000);
            return x + y;
        }
        void AddComplete(IAsyncResult iar)
        {
            Console.WriteLine($"Add() завершен в потоке {Thread.CurrentThread.ManagedThreadId}");
            AsyncResult ar = (AsyncResult)iar;
            MySample tmp = (MySample)ar.AsyncDelegate;
            int answer = tmp.EndInvoke(ar);
            Console.WriteLine($"Результат: 10 + 10 = {answer}");
            string msg = (string) iar.AsyncState; //сообщение
            Console.WriteLine(msg); //сообщение
            isDone = true;
        }
    }
}
```
## Параллельное программирование

Некоторые типы пространства System.Threading:
```csharp
Interlocker         Атомарные операции для переменных, разделяемых между несколькими потоками
Monitor             Синхронизация потоковых объектов, используя блокировки и ожидания/сигналы.
Mutex               Примитив синхронизации для синхронизации между доменами
Parameterized-ThreadStart   Делегат вызова метода с произвольным количеством аргументов
Semaphore           Тип ограничения количества потоков, имеющих доступ к ресурсу
Thread              Поток, выполняемый в среде CLR. Порождение новых потоков.
ThreadPool          Взаимодействие с поддерживаемым средой CLR пулом потоков
ThreadPriority      Уровень приоритета потока
ThreadStart         Указывание метода для вызова в заданном потоке, целевые методы должны иметь тот же прототип
ThreadState         Допустимые состояния потока
Timer               Механизм выполнения метода через указанные интервалы
TimerCallback       Тип делегата используется в сочетании с типами Timer
```

Некоторые статические члены класса Thread
```csharp
CurrentContext      Возвращает контекст, в котором выполняется поток
CurrentThread       Возвращает ссылку на текущий выполняемый поток
GetDomain()         Ссылка на текущий домен приложения, в котором выполняется поток
GetDomainID()       Идентификатор домена, в котором выполняется поток
Sleep()             Приостановление текущего потока на указанное время
```

Избранные члены экземпляры класса Thread
```csharp
IsAlive             Возвращает значение, что запищен ли поток
IsBackground        Значение, указывающее, является ли этот поток фоновым
Name                Дружественное текстовое имя потока
Priority            Приоритет потока, принимающий значение из перечисления ThreadPriority
ThreadState         Состояние потока, принимающий значение из ThreadState
Abort()             Указание на необходимость как можно более скорого прекращения работы потока
Interrupt()         Прерывает текущий поток на период ожидания
Join()              Блокирует вызывающий поток, пока указанный не заверщится
Resume()            Возобновляет выполнение ранее приостановленного потока
Start()             Указание на необходимость как можно более скорого запуска потока
Suspend()           Приостанавливает поток.    
```
Потоки могут быть переднего плана и фоновыми потоками. Преденего плана предохраняют текущее приложение от завершения, пока не завершат свою работу. Фоновые потоки (потоки-демоны) воспринимаются CLR как расширенные, которые могут быть игнорированы. Если все потоки переднего плана завершены, то фоновые автоматически уничтожаются.

Для того, чтобы поток был фоновым, нужно установить свойство IsBackground в истину.

Пример получения информации о потоке:
```csharp
public static void Main()
{
    Thread primaryThread = Thread.CurrentThread;
    primaryThread.Name = "ThePrimaryThread";
    Console.WriteLine($"Имя текущего домена: {Thread.GetDomain().FriendlyName}");
    Console.WriteLine($"Ид текущего контекста: {Thread.CurrentContext.ContextID}");
    Console.WriteLine($"Имя потока: {primaryThread.Name}");
    Console.WriteLine($"Запущенность потока: {primaryThread.IsAlive}");
    Console.WriteLine($"Приоритет потока: {primaryThread.Priority}");
    Console.WriteLine($"Состояние потока: {primaryThread.ThreadState}");
    Console.ReadLine();
}
```
Пример выполнения метода без параметра и аргументов во вторичном потоке:
```csharp
public static void Main()
{
    Thread primaryThread = Thread.CurrentThread;
    primaryThread.Name = "Поток";
    Console.WriteLine($"Работа Main() в потоке {Thread.CurrentThread.Name}");
    Thread backgrondThread = new Thread(new ThreadStart(MyPrint)); //не принимает аргументов и без параметров
    backgrondThread.Name = "Secondary";
    backgrondThread.Start();
    Console.WriteLine("Деланье дополнительной работы.");
    Console.ReadLine();
}
static void MyPrint()
{
    for (int i = 0; i < 10; i++)
    {
        Console.WriteLine($"Поток {Thread.CurrentThread.Name}: {i+1}");
        Thread.Sleep(1000);
    }
}
```
Пример выполнения метода c аргументами в ФОНОВОМ !!! потоке:
```csharp
class MyClass
{
    public int A { get; set; }
    public int B { get; set; }
    public MyClass(int a, int b)
    {
        this.A = a;
        this.B = b;
    }
}
public static void Main()
{
    Thread primaryThread = Thread.CurrentThread;
    primaryThread.Name = "Поток";
    Console.WriteLine($"Работа Main() в потоке {Thread.CurrentThread.Name}");
    MyClass myClass = new MyClass(2,2);
    Thread backgrondThread = new Thread(new ParameterizedThreadStart(MyPrint)); //не принимает аргументов и без параметров
    backgrondThread.Name = "Secondary";
    backgrondThread.IsBackground = true;
    backgrondThread.Start(myClass);
    Console.WriteLine("Деланье дополнительной работы.");
    Console.WriteLine("Нажать кнопку для завершения всех потоков");
    Console.ReadLine();
}
static void MyPrint(object data)
{
    if (data is MyClass myClass)
    {
        for (int i = 0; i < 20; i++)
        {
            Console.WriteLine($"Поток {Thread.CurrentThread.Name}: {myClass.A} + {myClass.B} = {myClass.A + myClass.B}");
            myClass.A++;
            Thread.Sleep(1000);
        }
    }
}
```
Пример выполнения метода c аргументами во вторичном потоке с ожиданием выполнения:
```csharp
//тот же самый класс MyClass
public static void Main()
{
    Thread primaryThread = Thread.CurrentThread;
    primaryThread.Name = "Поток";
    Console.WriteLine($"Работа Main() в потоке {Thread.CurrentThread.Name}");
    MyClass myClass = new MyClass(2,2);
    Thread backgrondThread = new Thread(new ParameterizedThreadStart(MyPrint)); //не принимает аргументов и без параметров
    backgrondThread.Name = "Secondary";
    backgrondThread.Start(myClass);
    Console.WriteLine("Деланье дополнительной работы.");
    waitHandle.WaitOne(); //подождать, пока не завершится другой поток
    Console.WriteLine("Ура! Завершен вторичный поток!");
    Console.ReadLine();
}
static void MyPrint(object data)
{
    if (data is MyClass myClass)
    {
        for (int i = 0; i < 5; i++)
        {
            Console.WriteLine($"Поток {Thread.CurrentThread.Name}: {myClass.A} + {myClass.B} = {myClass.A + myClass.B}");
            myClass.A++;
            Thread.Sleep(1000);
        }
    }
    waitHandle.Set();//уведомление другого потока что работа завершена
}
```
Пример выполения метода с импользованием пула потоков:
```csharp
//тот же самый класс MyClass
public static void Main()
{
    Console.WriteLine($"Работа Main() в потоке {Thread.CurrentThread.Name}");
    MyClass myClass = new MyClass(2,2);
    WaitCallback workItem = new WaitCallback(MyPrint); 
    for (int i = 0; i < 10; i++)
        ThreadPool.QueueUserWorkItem(workItem, myClass); //запуск метода в очередь 10 раз
    Console.WriteLine("Нажать кнопку для завершения всех потоков");
    Console.ReadLine();
}
static void MyPrint(object data)
{
    if (data is MyClass myClass)
    {
        for (int i = 0; i < 20; i++)
        {
            Console.WriteLine($"Поток {Thread.CurrentThread.Name}: {myClass.A} + {myClass.B} = {myClass.A + myClass.B}");
            myClass.A++;
            Thread.Sleep(1000);
        }
    }
}
```

## Синхронизация паралельно выполняемых потоков

При вызове методов в разных потоках с одинаковым используемым ресурсом могут быть получены неправильные результаты, так как один поток может вытеснить другой и ресурс может быть поврежден.

Некоторые статические члены класса System.Threading.Interlocked
```csharp
CompareExchange()   Безопасно проверяет два значения на равенство и, ежли они равны, то заменяет одно из значений третьим
Decrement()         Безопасно уменьшить значение на 1
Exchange()          Безопасно меняет два значения местами
Increment()         Безопасно увеличивает значение на 1
```

Пример использования маркера блокировки:
```csharp
static Random r = new Random();
class MyClass
{
    private object thredLock = new object(); //маркер блокировки
    public void Print()
    {
        lock (thredLock) //использование маркера блокировки
        {
            Console.WriteLine($"Поток \"{Thread.CurrentThread.Name}\" числа: ");
            for (int i = 0; i < 10; i++)
            {
                Console.Write($"{i + 1},");
                Thread.Sleep(1000 * r.Next(3));
            }
            Console.Write("конец\n");
        }
    }
}
public static void Main()
{
    MyClass myClass = new MyClass();
    Thread[] threads = new Thread[10]; //10 потоков
    for (int i = 0; i < 10; i++)
    {
        threads[i] = new Thread(new ThreadStart(myClass.Print));
        threads[i].Name = $"Поток № {i+1}";
    }
    foreach (var th in threads)
        th.Start();
    Console.ReadLine();
}
```
Пример класса MyClass выше с применением класса System.Threading.Monitor вместо lock():
```csharp
class MyClass
{
    private object threadLock = new object(); //маркер блокировки
    public void Print()
    {
        Monitor.Enter(threadLock);   
        try
        {
            Console.WriteLine($"Поток \"{Thread.CurrentThread.Name}\" числа: ");
            for (int i = 0; i < 10; i++)
            {
                Console.Write($"{i + 1},");
                Thread.Sleep(1000 * r.Next(3));
            }
            Console.Write("конец\n");
        }
        finally
        {
            Monitor.Exit(threadLock);
        }
    }
}
```
Пример синхронизации с использованием System.Threading.Interlocked:
```csharp
Interlocked.Increment(ref intVal); //увеличение на 1
Interlocked.Exchange(ref intVal, 55); //установка заданного значения
Interlocked.CompareExchange(ref intVal, 99, 0); //присвоить 99, если равно 0
```
Простая синхронизация атрибутом:
```csharp
[Synchronization] //теперь все методы безопасны к потокам
class MyClass : ContextBoundObject
{
    public void Print()
    {
        Console.WriteLine($"Поток \"{Thread.CurrentThread.Name}\" числа: ");
        for (int i = 0; i < 10; i++)
        {
            Console.Write($"{i + 1},");
            Thread.Sleep(1000 * r.Next(3));
        }
        Console.Write("конец\n");
    }
}
```
## Параллельное программирование TPL

Начиная с версии .NET 4.0 можно использовать библиотеку параллельных задач (TPL) System.Threading.Tasks для построения многопоточных приложений. Библиотека автоматом распределяет нарузку приложения между доступными процессорами в пуле потоков CLR. 

Пример приложения WPF с использованием TPL:
```csharp
//окно:
<Window x:Class="WpfApp1.MainWindow"
...
        Title="Пример TPL" Height="200" Width="400">
    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="*"/>
            <RowDefinition Height="Auto"/>
        </Grid.RowDefinitions>
        <Label Grid.Row="0" Grid.Column="0" Content="Пример приложения"/>
        <TextBox Grid.Row="1" Grid.Column="0" Margin="5,5,5,5"/>
        <Grid Grid.Row="2" Grid.Column="0">
            <Grid.ColumnDefinitions>
                <ColumnDefinition Width="Auto"/>
                <ColumnDefinition Width="*"/>
                <ColumnDefinition Width="Auto"/>
            </Grid.ColumnDefinitions>
            <Button Name="cmdCancel" Grid.Row="0" Grid.Column="0" Margin="5,5,5,5" Click="CmdCancel_OnClick" Content="Отмена"/>
            <Button Name="cmdProcess" Grid.Row="0" Grid.Column="2" Margin="5,5,5,5" Click="CmdProcess_OnClick" Content="Обработка!"></Button>
        </Grid>
    </Grid>
</Window>
public partial class MainWindow : Window
{
    private CancellationTokenSource cancelToken = new CancellationTokenSource(); //для отмены операции
    public MainWindow()
    {
        InitializeComponent();
    }
    private void CmdProcess_OnClick(object sender, RoutedEventArgs e)
    {
        Task.Factory.StartNew(() => ProcessFiles()); //вызов метода во вторичном потоке
    }
    private void ProcessFiles()
    {
        ParallelOptions parOpts = new ParallelOptions(); //опции для хранения токена отмены операции
        parOpts.CancellationToken = cancelToken.Token; //хран
        parOpts.MaxDegreeOfParallelism = Environment.ProcessorCount; //хран
        string[] files = Directory.GetFiles(@".\Test", "*.jpg",SearchOption.AllDirectories);
        string newDir = @".\MidifiedTest";
        Directory.CreateDirectory(newDir);
        try //отмена оперции путем отслеживания исключения
        {
            Parallel.ForEach(files, parOpts, curr =>
            {
                parOpts.CancellationToken.ThrowIfCancellationRequested(); //исключение если отмена операции
                string filename = Path.GetFileName(curr);
                using (Bitmap bitmap = new Bitmap(curr))
                {
                    bitmap.RotateFlip(RotateFlipType.Rotate180FlipNone);
                    bitmap.Save(Path.Combine(newDir, filename));
                    //this.Title = $"Выполенено {filename} в потоке {Thread.CurrentThread.ManagedThreadId}"; //не работает в многопоточной среде
                    Thread.Sleep(1000);
                    this.Dispatcher.Invoke(() => Debug.WriteLine($"Выполнено {filename}")); //безопасное в отношении потоков обновление интерфейса

                }
            });
            this.Dispatcher.Invoke(() => this.Title = $"Все Выполнено!"); //безопасное в отношении потоков обновление интерфейса
        }
        catch (OperationCanceledException ex)
        {
            this.Dispatcher.Invoke(() => Debug.WriteLine(ex.Message));
        }
    }
    private void CmdCancel_OnClick(object sender, RoutedEventArgs e)
    {
        cancelToken.Cancel(); //для отмены операции
    }
}
```
Пример простого вызова асинхронной задачи с помощью метода Parallel.Invoke():
```csharp
public static void Main()
{
    GetBook();
    Console.WriteLine("Нажмите любую кнопку");
    Console.ReadLine();
}
static void GetBook()
{
    WebClient client = new WebClient();
    client.DownloadStringCompleted += (sender, args) =>
    {
        Console.WriteLine("Загрузка завершена.");
        //string[] words = {"one","gooolo","there","most","twenty","moogo","one","one","gooolo","gooolo","most"};
        string[] words = args.Result.Split(new char[]{' ',',','.',';',':','-','?','/','\u000A'}, StringSplitOptions.RemoveEmptyEntries); //получение слов
        Parallel.Invoke(
            () =>
            {
                var tenMost = words
                    .Where(w => w.Length >= 3)
                    .GroupBy(w => w)
                    .OrderByDescending(g => g.Count())
                    .Select(g=>g.Key)
                    .Take(10)
                    .ToArray();
                Console.WriteLine("Список наиболее попадающихся слов:");
                foreach (var w in tenMost)
                {
                    Console.WriteLine($"{w}");
                }
            },
            () =>
            {
                var longest = words.OrderByDescending(w => w.Length).Select(w => w).FirstOrDefault();
                Console.WriteLine($"Самoе длинное слово: {longest}");
            }
            ); //параллельно выполняемые действия
    }; //это выполнится по завершении загрузки
    client.DownloadStringAsync(new Uri(@"http://www.gutenberg.org/files/98/98-8.txt")); //загрузка в асинхронном режиме
}
```
## Параллельный PLINQ

Некоторые члены класса ParallelEnumerable из пространства System.Linq:
```csharp
AsParallel()                Указание, что остаток запроса должен быть по возможности распараллелен
WithCalcellation()          Указание, что PLINQ должен периодически отслеживать состояние признака отмены и при необходимости отменять выполенение
WithDegreeOfParallelism()   Указание максимального количества процессоров, которое PLINQ должна задействовать
ForAll()                    Обработка результатаов параллельно без предварительноо слияния с потоком потребителя, до foreach
```

Пример использования PLINQ:
```csharp
private static CancellationTokenSource cancelToken = new CancellationTokenSource(); //прерывание выполенения
static void Main()
{
    Console.WriteLine("Нажмите любую кнопку для Начала");
    Console.ReadKey();
    Console.WriteLine("Выполнение");
    Task.Factory.StartNew(() => MyCalcInts());
    Console.WriteLine("Введите Q для прерывания задачи");
    string answer = Console.ReadLine();
    if (answer.Equals("Q", StringComparison.OrdinalIgnoreCase))
        cancelToken.Cancel(); //прерывание выполнения
    Console.WriteLine("Нажмите любую кнопку для выхода");
    Console.ReadKey();
}
static void MyCalcInts()
{
    int[] source = Enumerable.Range(1, 100_000_000).ToArray();
    try
    {
        int[] modInts = source
            .AsParallel() //паралельное выполнение
            .WithCancellation(cancelToken.Token) //с прерыванием паралельного выполения
            .Where(n => n % 2 == 0)
            .OrderByDescending(n => n)
            .Select(n => n)
            .ToArray();
        int countEvens = modInts.Count();
        Console.WriteLine($"Найдено {countEvens} четных чисел");
    }
    catch (OperationCanceledException ex)
    {
        Console.WriteLine(ex.Message);
    }
}
```


## Вызовы методов через регулярные интервалы времени - Timer

Тип имеет связанный делегат TimerCallback. 
```csharp
static void PrintTime(object state)
{
    Console.WriteLine($"Время: {DateTime.Now.ToLongTimeString()} Сообщение: {state.ToString()}");
}
public static void Main()
{
    TimerCallback tc = new TimerCallback(PrintTime);
    var _ = new Timer(tc,"Привет!",0,1000); //отбрасывание
    Console.WriteLine("Нажмите любую кнопку...");
    Console.ReadLine();
}
```











