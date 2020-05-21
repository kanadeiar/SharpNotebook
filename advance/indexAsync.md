# Асинхронное, параллельное, многопоточние программирование

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
## Синхронизация паралельно выполняемых потоков

При вызове методов в разных потоках с одинаковым используемым ресурсом могут быть получены неправильные результаты, так как один поток может вытеснить другой и ресурс может быть поврежден.

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


## Асинхронное программирование async/await















