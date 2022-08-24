# Асинхронные вычисления

К вычислительным асинхронным операциям, в частности, относятся компиляция кода, проверка орфографии, проверка грамматики, пересчет электронных таблиц, перекодирование аудио- и видеоданных, создание миниатюр изображений.

Для асинхронных вычисления применяют прежде всего пул потоков.

## Простые вычислительные операции

Для добавления в очередь пула потоков асинхронных вычислительных операций можно вызвать методов класса ThreadPool - QueueUserWorkItem(). Этот метод ставит «рабочий элемент» вместе с дополнительными данными состояния в очередь пула потоков и сразу возвращают управление приложению.

Пример простейшего использования пула потоков:

```csharp
static void Method(object? obj)
{
    Console.WriteLine("Данные на входе метода: {0}", obj);
    Thread.Sleep(3000);
}
// вызов
ThreadPool.QueueUserWorkItem(Method, "объект");
Thread.Sleep(5000);
```

Планировщик Windows решает, какой поток должен выполняться первым, или же планирует их для одновременного выполнения на многопроцессорном компьютере.

## Контексты исполнения потоков

С каждым потоком связан определенный контекст исполнения. Он включает в себя параметры безопасности, хоста и данные логического вызова. Когда поток исполняет код, значения параметров контекста исполнения оказывают влияние на некоторые операции. По умолчанию CLR автоматически копирует контекст исполнения самого первого потока во все вспомогательные потоки. Сбор всей информации и ее копирование во вспомогательные потоки занимает немало времени, но зато гарантируется безопасность.

Класс ExecutionContext в пространстве имен System.Threading позволяет управлять копированием контекста исполнения потока.

## Скоординированная отмена

Платформа .NET предлагает стандартный паттерн операций отмены. Этот паттерн является скоординированным (cooperative), то есть требует явной поддержки отмены операций. Для отмены длительных вычислительных операций можно применять специальные классы CancellationTokenSource и CancellationToken.

Экземпляр CancellationToken относится к упрощенному значимому типу, так как содержит всего одно закрытое поле: ссылку на свой объект CancellationTokenSource. Цикл вычислительной операции может периодически обращаться к свойству IsCancellationRequested объекта CancellationToken, чтобы узнать, не требуется ли раннее завершение его работы, то есть прерывание операции.

Пример использования:

```csharp
static void Test(CancellationToken token, int value)
{
    var val = 0;
    for (int i = 0; i < value; i++)
    {
        if (token.IsCancellationRequested)
        {
            Console.WriteLine("Отменено");
            break;
        }
        val += i;
        Thread.Sleep(1000);
        Console.WriteLine(val);
    }
    Console.WriteLine("Готово: {0}!", val);
}
// использование токена отмены асинхронной операции
var cts = new CancellationTokenSource();
ThreadPool.QueueUserWorkItem(_ => Test(cts.Token, 100));
Console.WriteLine("Любую кнопку нажать для отмены");
Console.ReadKey();
cts.Cancel();
```

Можно создать новый объект CancellationTokenSource, связав друг с другом другие объекты CancellationTokenSource. Отмена этого нового объекта произойдет при отмене любого из входящих в его состав объектов.

Можно отменить операцию по истечении определенного времени - для этого нужно передать определенный параметр в конструктор класса отмены операций или использовать метод CancelAfter.

```csharp
var cts = new CancellationTokenSource();
cts.CancelAfter(1000);
```

## Задания

Для обхода многих ограничений и недостатков прямого использования методов пула потоков было введено понятие заданий (tasks), выполнение которых осуществляется посредством типов из пространства имен System.Threading.Tasks. 

Все вместе типы из пространства System.Threading.Tasks называются библиотекой параллельных задач (Task Parallel Library — TPL). С помощью типов из System.Threading.Tasks можно строить мелкомодульный масштабируемый параллельный код без необходимости напрямую иметь дело с потоками или пулом потоков. Библиотека TPL будет автоматически распределять нагрузку приложения между доступными процессорами в динамическом режиме с применением пула потоков исполняющей среды. Библиотека TPL поддерживает разбиение работы на части, планирование потоков, управление состоянием и другие низкоуровневые детали. В конечном итоге появляется возможность максимизировать производительность приложений .NET Core, не сталкиваясь со сложностями прямой работы с потоками.

Полностью аналогичные операции постановки задачи в пул потоков, в итоге превращающиеся в одинаковый IL код:

```csharp
ThreadPool.QueueUserWorkItem(Method, 5);
new Task(Method, 5).Start();
Task.Run(() => Method(5)); //наилудший вариант
```

При желании конструктору можно передавать флаги из перечисления TaskCreationOptions, управляющие способами выполнения заданий. Флаги конструктора:

None - без изменения

PreferFairness - как можно скорее поставить задание на выполнение

LongRunning - создавать потоки в пуле потоков

AttachedToParent - присоединяет задание к его родителю

DenyChildAttach - задача интерпретируется как обычная, а не дочерняя

HideScheduler - дочерние задачи будут использовать стандартный планировкик вместо родительского

### Структура задания

Каждый объект Task состоит из набора полей, определяющих состояние задания:

- идентификатор

- состояение выполнения задания

- ссылка на родительское задание

- ссылка на объект с временем создания задания

- ссылка на метод обратного вызова

- ссылка на класс ExecutionContext

- ссылка на объект ManualResetEventSlim

Каждый объект Task имеет ссылку на дополнительное состояние, создаваемое по требованию. Это дополнительное состояние включает в себя объект CancellationToken, коллекцию объектов ContinueWithTask, коллекцию объектов
Task для дочерних заданий, ставших источником необработанных исключений, и прочее. Все это ест ресурсы. Если дополнительные возможности класса Task не нужны, для более эффективного расходования ресурсов рекомендуем воспользоваться методом ThreadPool.QueueUserWorkItem.

Узнать, на какой стадии своего жизненного цикла находится задание, можно при помощи предназначенного только для чтения свойства Status объекта Task.

Флаги:

Created - задание создано вручную и готово к запуску

WaitingForActivation - задание создано неявно и запускается автоматически

WaitingToRun - задание запланировано

Running - задание выполняется

WaitingForChildrenToComplete - задание ждет завершения дочерних заданий

RanToCompletion - успешно выполнено

Canceled - отменено

Faulted - ошибка

Объект Task оказывается в состоянии WaitingForActivation, если он создается при помощи одной из следующих функций: ContinueWith, ContinueWhenAll, ContinueWhenAny или FromAsync. Задание, созданное путем конструирования
объекта TaskCompletionSource<TResult>, также оказывается в состоянии WaitingForActivation. Это состояние означает, что планирование задания управляется его собственной инфраструктурой.

### Получение результата задания

Можно дождаться завершения задания и после этого получить результат его выполнения.

Можно создать объект Task<TResult> (производный от объекта Task) и в качестве универсального аргумента TResult передать тип результата, возвращаемого вычислительной операцией. Затем остается дождаться завершения выполняющегося задания и получить результат.

Если вычислительное задание генерирует необработанное исключение, оно поглощается и сохраняется в коллекции, а потоку пула разрешается вернуться в пул. Затем при вызове метода Wait или свойства Result эти члены вбросят исключение System.AggregateException. Сам тип AggregateException инкапсулирует коллекцию исключений.

Можно ожидать завершения не только одного задания, но и массива объектов Task. Метод WaitAny блокирует вызов потоков до завершения выполнения всех объектовв массиве Task. Статический метод WaitAll класса Task блокирует вызывающий поток до завершения всех объектов Task в массиве. Отмена же метода посредством структуры CancellationToken приводит к исключению OperationCanceledException.

Для отмены задания можно воспользоваться объектом CancellationTokenSource.

Пример запуска простого метода асинхронно и получения от него результата:

```csharp
static int Sum(int values)
{
    var sum = 0;
    for (; values > 0; values--)
    {
        sum += values;
    }
    return sum;
}
// запуск и получение результата
var task = new Task<int>(x => Sum((int)x!), 100);
task.Start();
task.Wait();
Console.WriteLine("Результат: {0}", task.Result);
```

Более сложный пример с отменой операции:

```csharp
static int Sum(int values, CancellationToken? token = null)
{
    var sum = 0;
    for (; values > 0; values--)
    {
        token?.ThrowIfCancellationRequested();
        sum += values;
    }
    return sum;
}
// запуск и получение результата
CancellationTokenSource cts = new CancellationTokenSource();
var task = new Task<int>(() => Sum(100, cts.Token), cts.Token);
task.Start();
cts.Cancel();
try
{
    Console.WriteLine("Результат: {0}", task.Result);
}
catch (AggregateException ex)
{
    ex.Handle(e => e is OperationCanceledException);
    Console.WriteLine("Операция отменена");
}
```

### Запуск нового задания после предидущего

Для написания масштабируемого программного обеспечения следует избегать блокировки потоков (Wait и Result).

Пример как только задание, выполняющее метод Sum, завершится, оно инициирует выполнение следующего задания (также на основе потока из пула), которое выведет результат.

```csharp
// пример запуска нового задания после предидущего
var task = new Task<int>(() => Sum(100, CancellationToken.None));
task.Start();
var cwt = task.ContinueWith(x => Console.WriteLine($"Сумма: {0}", task.Result), TaskContinuationOptions.None);
```

В метод ContinueWith можно передавать флаги:

None - без изменения

PreferFairness - как можно скорее поставить задание на выполнение

LongRunning - создавать потоки в пуле потоков

AttachedToParent - примоединяет задание к родителю

DenyChildAttach - задача интерпретируется как обычная, а не дочерняя

HideScheduler - дочерние задачи будут использовать стандартный планировкик вместо родительского

LazyCancellation - запрещает отмену до завершения предшественника

ExecuteSynchronously - если первое задание уже завершено, поток, вызывающий ContinueWith, выполняет задание ContinueWith

NotOnRanToCompletion, NotOnFaulted, NotOnCanceled - когда запускать задание

```csharp
// пример различных вариантов
var task = Task.Run(() => Sum(100, CancellationToken.None));
task.ContinueWith(x => Console.WriteLine($"Сумма: {0}", task.Result), TaskContinuationOptions.OnlyOnRanToCompletion);
task.ContinueWith(x => Console.WriteLine($"Ошибка: {0}", task.Exception), TaskContinuationOptions.OnlyOnFaulted);
task.ContinueWith(x => Console.WriteLine($"Отмена"), TaskContinuationOptions.OnlyOnCanceled);
```

### Дочерние задания

Задания также поддерживают отношения предок-потомок. Родительское задание может создавать дочерние задания и запускать их.

```csharp
// пример применения дочерних заданий
var parent = new Task<int[]>(() =>
{
    var results = new int[3];
    new Task(() => results[0] = Sum(100), TaskCreationOptions.AttachedToParent).Start();
    new Task(() => results[1] = Sum(200), TaskCreationOptions.AttachedToParent).Start();
    new Task(() => results[2] = Sum(300), TaskCreationOptions.AttachedToParent).Start();
    return results;
});
var cwt = parent.ContinueWith(x => Array.ForEach(x.Result, Console.WriteLine));
parent.Start();
```

### Фабрики заданий

Можно создать фабрику заданий, которая будет инкапсулировать нужное состояние заданий и выдавать их набор. Для создания группы заданий, не возвращающих значений, конструируетсяикласс TaskFactory. Если же эти задания должны возвращать некое значение, потребуется класс TaskFactory<TResult>, которому в обобщенном аргументе TResult передается желаемый тип возвращаемого значения.

Пример применения фабрики:

```csharp
var parent = new Task(() =>
{
    var cts = new CancellationTokenSource();
    var tf = new TaskFactory<int>(cts.Token, TaskCreationOptions.AttachedToParent, TaskContinuationOptions.ExecuteSynchronously, TaskScheduler.Default);
    var childs = new Task<int>[]
    {
        tf.StartNew(() => Sum(100, cts.Token)),
        tf.StartNew(() => Sum(200, cts.Token)),
        tf.StartNew(() => Sum(150, cts.Token)),
    };
    for (int i = 0; i < childs.Length; i++)
    {
        childs[i].ContinueWith(x => cts.Cancel(), TaskContinuationOptions.OnlyOnFaulted);
    }
    tf.ContinueWhenAll(childs, x => x.Where(t => !t.IsFaulted && !t.IsCanceled).Max(t => t.Result), CancellationToken.None)
    .ContinueWith(t => Console.WriteLine("Максимальное: {0}", t.Result), TaskContinuationOptions.ExecuteSynchronously);
});
parent.ContinueWith(p =>
{
    var sb = new StringBuilder("Обнаружены исключения:" + Environment.NewLine);
    foreach (var e in p.Exception.Flatten().InnerExceptions)
        sb.AppendLine(" " + e.GetType().ToString());
    Console.WriteLine(sb.ToString());
}, TaskContinuationOptions.OnlyOnFaulted);
parent.Start();
```

### Планировщики заданий

Задания обладают очень гибкой инфраструктурой. Для получения ссылки на планировщик заданий в пуле потоков используется статическое свойство Default класса TaskScheduler.

Планировщики заданий контекста синхронизации обычно применяются в приложениях настольных (Windows Forms, WPF) и мобильных (Xamarin).

Они планируют задания в потоке графического интерфейса приложения. Получить на него ссылку можно с помощью cтатического метода FromCurrentSynchronizationContext класса TaskScheduler.

Пример использования:

```csharp
private CancellationTokenSource _cts;
private readonly TaskScheduler _syncContextTaskScheduler;
private string _Text;
public string Text
{
    get => _Text;
    set => Set(ref _Text, value);
}
public MainWindowViewModel()
{
    _syncContextTaskScheduler =
    TaskScheduler.FromCurrentSynchronizationContext();
    Text = "Тестирование";
}
private void OnTestCommandExecuted(object p)
{
    if (_cts != null)
    { // Операция начата, отменяем ее
        _cts.Cancel();
        _cts = null;
    }
    else
    { // Операция не начата, начинаем ее
        Text = "Операция запускается";
        _cts = new CancellationTokenSource();
        Task<int> t = Task.Run(() => Sum(20000, _cts.Token), _cts.Token);
        t.ContinueWith(task => Text = "Результат: " + task.Result,
        CancellationToken.None,
        TaskContinuationOptions.OnlyOnRanToCompletion,
        _syncContextTaskScheduler);
        t.ContinueWith(task => Text = "Операция отменена",
        CancellationToken.None, TaskContinuationOptions.OnlyOnCanceled,
        _syncContextTaskScheduler);
        t.ContinueWith(task => Text = "Ошибка операции",
        CancellationToken.None, TaskContinuationOptions.OnlyOnFaulted,
        _syncContextTaskScheduler);
    }
}
```

### Класс Parallel

Для упрощения программирования с применением заданий созданы специальные методы в классе System.Threading.Tasks.Parallel. Он содержит методы , которые позволяют осуществлять итерацию по коллекции данных (точнее по объекту, реализующему интерфейс IEnumerable<T>)в параллельной манере. Максимальный выигрыш от этого инструмента достигается при наличии множество элементов для обработки или когда обработка каждого элемента представляет собой длительную вычислительную операцию.

Понадобится применять делегаты System.Func<T> и System.Action<T> для указания целевого метода, который будет вызываться при обработке данных. Работа с этим классом упрощается за счет использования подходящих анонимных методов или лямбда-выражений С#.

Примеры использования этих методов:

```csharp
private static void DoWork(int i)
{
    Console.WriteLine(i);
}
//перебор
Parallel.For(0, 1000, x => DoWork(x));
Parallel.ForEach(collection, x => DoWork(x));
Parallel.Invoke(() => DoWork(1), () => DoWork(2));
```

Методы класса Parallel потребляют много ресурсов — приходится выделять память под делегаты, которые вызываются по одному для каждого рабочего элемента.

Существуют перегруженные версии и для методов For и ForEach, позволяющие передавать три делегата:

- Делегат локальной инициализации задания (localInit) для каждой выполняемой задачи вызывается только один раз — перед получением команды на обслуживание рабочего элемента.

- Делегат body вызывается один раз для каждого элемента, обслуживаемого участвующими в процессе потоками.

- Делегат локального состояния каждого потока (localFinally) вызывается один раз для каждого задания, после того как оно обслужит все переданные ему рабочие элементы. Также он вызывается, если код делегата body становится источником необработанного исключения.

Пример использования таких версий методов параллельного перебора коллекций - подсчет количества символов:

```csharp
var names = new string[] { "Паша", "Петя", "Соня" };
int total = 0;
var result = Parallel.ForEach<string, int>(
    names,
() =>
{
    return 0;
},
(name, loopState, index, taskLocalTotal) =>
{
    var len = name.Length;
    return taskLocalTotal + len;
},
taskLocalTotal =>
{
    Interlocked.Add(ref total, taskLocalTotal);
});
System.Console.WriteLine("Всего символов: {0}", total);
```

Результат работы цикла можно определить при помощи свойств. Если свойство IsCompleted возвращает значение true, значит, цикл пройден полностью, и все элементы обработаны. Если свойство IsCompleted возвращает значение false, а свойство LowestBreakIteration — значение null, значит, каким-то из потоков был вызван метод Stop. Если же в последнем случае значение, возвращаемое свойством LowestBreakIteration, отлично от null, значит, каким-то из потоков был вызван метод Break.

### Параллельный LINQ

Повысить производительность LINQ можно при помощи языка параллельных запросов (Parallel LINQ), позволяющего
последовательный запрос превратить в параллельный (parallel query). Внутри задействуются задания, распределяемые по потокам. Максимальный выигрыш от этого инструмента достигается при наличии множество элементов для обработки или когда обработка каждого элемента представляет собой длительную вычислительную операцию.

Для вызова параллельных версий методов LINQ следует преобразовать последовательный запрос (основанный на интерфейсе IEnumerable или IEnumerable<T>) в параллельный (основанный на классе ParallelQuery или ParallelQuery<T>), воспользовавшись методом расширения AsParallel класса ParallelEnumerable. Переключиться обратно с параллельного режима на последовательный можно при помощи метода AsSequential класса ParallelEnumerable.

Пример применения Parallel LINQ:

```csharp
var names = new string[] { "test", "zero", "hero", "one", "three" };
var pquery = names.AsParallel(); // переход в параллельный режим
pquery = pquery.Select(x => $"Имя: {x}");
var count = 0;
pquery.ForAll(x => count += 1);
System.Console.WriteLine("Количество: {0} шт.", count);
var query = pquery.AsSequential().OrderBy(x => x); // переход обратно в последовательный режим
foreach (var e in query)
{
    Console.WriteLine(e);
}
```

Операторы, предназначенные для выполнения неупорядоченных операций: Distinct, Except, Intersect, Union, Join,GroupBy, GroupJoin и ToLookup. После любого из этих операторов нужно вызвать метод AsOrdered для упорядичивания элементов.

Parallel LINQ также предоставляет дополнительные методы класса ParallelEnumerable, позволяющие управлять обработкой запросов: 

- WithCancellation - возможность в любой момент остановить обработку запроса, 

- WithDegreeOfParallelism - задает максимальное количество потоков, 

- WithExecutionMode - позволяет задать принудительную обработку в параллельном режиме, 

- WithMergeOptions - управление буферизацией и слиянием элементов.

### Периодически выполняемые методы

Класс Timer, определенный в пространстве имен System.Threading позволяет с заданной периодичностью вызывать методы с использованием пула потоков.

У этого класа есть несколько перегруженных конструкторов.

В параметрах которых нужно указать метод, который будет выполняться потоком из пула. Метод должен соответствовать типу делегата:

```csharp
delegate void TimerCallback(Object state);
```

Еще в параметрах нужно передать state - служит для передачи методу обратного вызова данных состояния.

Параметр dueTime позволяет задать для CLR время ожидания (в миллисекундах) перед первым вызовом метода обратного вызова.

Последний параметр period указывает периодичность (в миллисекундах) последующих обращений к методу обратного вызова.

В пуле имеется всего один поток для всех объектов Timer. Именно он знает время активизации следующего таймера. В этот момент поток пробуждается и вызывает метод QueueUserWorkItem объекта ThreadPool, чтобы добавить в очередь пула потоков элемент, активизирующий метод обратного вызова.
 
Можно использовать метод Change() для указания нового времени срабатывания таймера и периодичности срабатываний. При указании периодичности срабатывания Timeout.Infinite(-1) метод вызывается только один раз.

Пример использования таймера - метод вызывается немедленно, а затем периодически каждые три:

```csharp
private static Timer _timer;
...
_timer = new Timer(Method, null, Timeout.Infinite, Timeout.Infinite);
_timer.Change(0, Timeout.Infinite);
static void Method(object? state)
{
    Console.WriteLine("Срабатывание таймера {0}", DateTime.Now);
    Thread.Sleep(100); //полезная работа
    _timer.Change(2000, Timeout.Infinite);
}
```

Пример реализации таймера с на основе асинхронного метода:

```csharp
private static async void DelayTimer()
{
    while (true)
    {
        Console.WriteLine("Срабатывание таймера: {0}", DateTime.Now);
        await Task.Delay(1000);
    }
}
//вызов
DelayTimer();
```









