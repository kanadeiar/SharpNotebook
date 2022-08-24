# Гибридная синхронизация потоков

На основе простейших конструкций синхронизации потоков можно строить более сложные конструкции синхронизации, где различные виды простейших конструкций скомбинированы.

## Простая гибридная конструкция

Пример простой гибридной конструкции:

```csharp
sealed class SimpleHybridLock : IDisposable
{
    // int используется примитивными конструкциями пользовательского режима (Interlocked-методы)
    private int _waiters = 0;
    // примитивная конструкция режима ядра
    private AutoResetEvent _waiterLock = new AutoResetEvent(false);
    public void Enter()
    {
        // Поток хочет получить блокировку
        if (Interlocked.Increment(ref _waiters) == 1)
            return; // Блокировка свободна, конкуренции нет, возвращаем управление
        // Блокировка захвачена другим потоком (конкуренция), приходится ждать.
        _waiterLock.WaitOne(); // Значительное снижение производительности
    }
    public void Leave()
    {
        // Поток освобождает блокировку
        if (Interlocked.Decrement(ref _waiters) == 0)
            return; // Другие потоки не заблокированы, возвращаем управление
        // Другие потоки заблокированы, пробуждаем один из них
        _waiterLock.Set(); // Значительное снижение производительности
    }
    public void Dispose() => _waiterLock.Dispose();
}
// использование
using (var locker = new SimpleHybridLock())
{
    locker.Enter();
    _value += i;
    locker.Leave();
}
```

## Зацикливание

Так как переходы в ядро значительно снижают производительность, а потоки остаются запертыми очень короткое время, общую производительность приложения можно повысить, заставив поток перед переходом в режим ядра на некоторое время зациклиться в пользовательском режиме. Если в это время блокирование, которого ожидает поток, станет возможным, переход в режим ядра не понадобится.

С помощью нетривиальной логики можно реализовать гибридное блокирование, предполагающее одновременно зацикливание, владение потоком и рекурсию.

Пример:

```csharp
public class AnotherHybridLock : IDisposable
{
    private int _waiters = 0;
    private AutoResetEvent _waiterLock = new AutoResetEvent(false);
    private int _spinCount = 4000;
    private int _owningThreadId = 0, _recursion = 0;
    public void Enter()
    {
        var threadId = Thread.CurrentThread.ManagedThreadId;
        if (threadId == _owningThreadId) { _recursion++; return; }
        var spinwait = new SpinWait();
        for (int spinCount = 0; spinCount < _spinCount; spinCount++)
        {
            if (Interlocked.CompareExchange(ref _waiters, 1, 0) == 0) goto GotLock;
            spinwait.SpinOnce();
        }
        if (Interlocked.Increment(ref _waiters) > 1)
        {
            _waiterLock.WaitOne(); // Значительное падение производительности
        }
    GotLock:
        _owningThreadId = threadId;
        _recursion = 1;
    }
    public void Leave()
    {
        var threadId = Thread.CurrentThread.ManagedThreadId;
        if (threadId != _owningThreadId)
            throw new SynchronizationLockException("Lock not owned by calling thread");
        if (--_recursion > 0) return;
        _owningThreadId = 0;
        if (Interlocked.Decrement(ref _waiters) == 0)
            return;
        _waiterLock.Set(); // Значительное падение производительности
    }
    public void Dispose() => _waiterLock.Dispose();
}
```

## Гибридные конструкции BCL

В FCL существует множество гибридных конструкций, использующих изощренную логику, которая должна удержать потоки в пользовательском режиме, повышая производительность приложения. В некоторых гибридных конструкциях возникновения конкуренции между потоками обращения к конструкциям режима ядра также не происходит. В результате, если конкуренция так и не возникает, приложению не приходится сталкиваться с падением производительности и необходимостью выделять память для объекта.

### Классы ManualResetEventSlim и SemaphoreSlim

Они функционируют точно так же, как их аналоги режима ядра, отличаясь только зацикливанием в пользовательском режиме, а также тем, что не создают конструкций ядра до возникновения конкуренции. Их методы Wait позволяют передать информацию о времени ожидания и объект CancellationToken.

Классы:

```csharp
public class ManualResetEventSlim : IDisposable
{
    public ManualResetEventSlim(bool initialState, int spinCount);
    public void Dispose();
    public void Reset();
    public void Set();
    public bool Wait(
    int millisecondsTimeout, CancellationToken cancellationToken);
    public bool IsSet { get; }
    public int SpinCount { get; }
    public WaitHandle WaitHandle { get; }
}
public class SemaphoreSlim : IDisposable
{
    public SemaphoreSlim(int initialCount, int maxCount);
    public void Dispose();
    public int Release(int releaseCount);
    public bool Wait(Int32 millisecondsTimeout, CancellationToken cancellationToken);
    public Task<bool> WaitAsync(int millisecondsTimeout,
    CancellationToken cancellationToken);
    public int CurrentCount { get; }
    public WaitHandle AvailableWaitHandle { get; }
}
```

### Класс Monitor

Самой популярной из гибридных конструкций синхронизации потоков является класс Monitor, обеспечивающий взаимоисключающее блокирование с зацикливанием, владением потоком и рекурсией.

Данная конструкция используется чаще других потому, что является одной из самых старых, для ее поддержки в C# существует встроенное ключевое слово, с ней по умолчанию умеет работать JIT-компилятор, а CLR пользуется ею от имени приложения.

С каждым объектом в куче может быть связана структура данных, называемая блоком синхронизации (sync block). Этот блок содержит поля: поле для объекта ядра, идентификатора потока-владельца, счетчика рекурсии и счетчика ожидающих потоков.

Класс Monitor является статическим, и его методы принимают на вход ссылки на любой объект кучи. Управление полями эти методы осуществляют в блоке синхронизации заданного объекта.

Методы класса:

```csharp
public static class Monitor
{
    public static void Enter(object obj);
    public static void Exit(object obj);
    // Можно также указать время блокирования (требуется редко):
    public static bool TryEnter(object obj, int millisecondsTimeout);
    public static void Enter(object obj, ref bool lockTaken);
    public static void TryEnter(object obj, int millisecondsTimeout, ref bool lockTaken);
}
```

В момент конструирования объекта этому индексу присваивается значение –1, что означает отсутствие ссылок на блок синхронизации. Затем при вызове метода Monitor.Enter CLR обнаруживает в массиве свободный блок синхронизации и присваивает ссылку на него объекту.

Метод Exit проверяет наличие потоков, ожидающих блока синхронизации. Если таких потоков не обнаруживается, метод возвращает индексу значение –1, означающее, что блоки синхронизации свободны и могут быть связаны с какими-нибудь другими объектами.

Образец использования этого класса:

```csharp
sealed class Transaction
{
    private readonly Object m_lock = new Object(); // Блокирование в рамках каждой транзакции ЗАКРЫТО
    private DateTime _lastDate;
    public void DoTransaction()
    {
        Monitor.Enter(m_lock);
        _lastDate = DateTime.Now;
        Monitor.Exit(m_lock);
    }
    public DateTime GetLast
    {
        get
        {
            Monitor.Enter(m_lock);
            DateTime temp = _lastDate;
            Monitor.Exit(m_lock);
            return temp;
        }
    }
}
// использование
var trans = new Transaction();
trans.DoTransaction();
var dattime = trans.GetLast;
```

C# есть упрощенный синтаксис в виде ключевого слова lock.

Пример:

```csharp
private void SomeMethod()
{
    lock (this)
    {
        // Этот код имеет эксклюзивный доступ к данным...
    }
}
```

Компилятором превращается в блок try - finally с использованием методов класса Monitor.

Но у него есть недостатки. Monitor.Exit вызывается в блоке finally. Вход в блок try и выход из блока снижает производительность метода.

### Класс ReaderWriterLockSlim

Управление потоками осуществляется следующим образом:

- Если один поток осуществляет запись данных, все остальные потоки, требующие доступа, блокируются.

- Если один поток читает данные, все остальные потоки, требующие доступа, продолжают работу, блокируются только потоки, ожидающие доступа на запись.

- После завершения работы потока, осуществлявшего запись данных, разблокируется либо один поток, ожидающий доступ на запись, либо все потоки, ожидающие доступ на чтение. При отсутствии заблокированных потоков блокировку получает следующий поток чтения или записи, которому это потребуется.

- После завершения всех потоков, осуществлявших чтение данных, разблокируется поток, ожидающий разрешения на запись. При отсутствии заблокированных потоков блокировку получит следующий поток чтения или записи, которому это потребуется.

## Выводы

Рекомендуется по возможности избегать кода, блокирующего потоки. Выполняя асинхронные вычисления или операции ввода-вывода, передавайте данные от одного потока к другому так, чтобы исключить одновременную попытку доступа к данным со стороны нескольких потоков. Если это невозможно, нужно стараться использовать методы классов Volatile и Interlocked, так как они работают быстро и не блокируют потоки.

Две основные причины, по которым приходится блокировать потоки:

- Упрощение модели программирования.

- Поток имеет определенное назначение.

## Классы коллекций для параллельного доступа

В BCL существует четыре безопасных в отношении потоков класса коллекций, принадлежащих пространству имен System.Collections.Concurrent: ConcurrentQueue, ConcurrentStack, ConcurrentDictionary и ConcurrentBag.

ConcurrentQueue -  Обработка элементов по алгоритму FIFO

ConcurrentStack - Обработка элементов по алгоритму LIFO

ConcurrentBag - Несортированный набор элементов с возможностью хранения дубликатов

ConcurrentDictionary - Несортированный набор пар ключ/значение

Эти классы коллекций являются неблокирующими. При попытке извлечь несуществующий элемент поток немедленно возвращает управление, а не блокируется, ожидая появления элемента. Именно поэтому такие методы, как TryDequeue, TryPop, TryTake и TryGetValue, при получении элемента возвращают значение true, а при его невозможности — false.

Для классов ConcurrentStack, ConcurrentQueue и ConcurrentBag метод GetEnumerator создает снимок содержимого коллекции и возвращает за фиксированные элементы; при этом реальное содержимое коллекции уже может измениться.

Свойство Count возвращает количество элементов в коллекции на момент запроса.

