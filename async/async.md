# Асинхронная синхронизация потоков

Класс SemaphoreSlim реализует эту идею в своем методе WaitAsync. Сигнатура самой сложной перегруженной версии этого метода выглядит так:

```csharp
public Task<bool> WaitAsync(int millisecondsTimeout, CancellationToken token);
```

Этим методом можно синхронизировать доступ к ресурсу в асинхронном режиме.

```csharp
static class SimpleStatic
{
    private static int _value;
    public static int Value => _value;
    public static async Task AddAsync(SemaphoreSlim? asyncLock)
    {
        await asyncLock!.WaitAsync(); // Запрос монопольного доступа к ресурсу
        _value++; // Работа с ресурсом
        asyncLock.Release(); // Снятие блокировки, чтобы ресурс стал доступен другим потокам
    }
}
// использование
using var semaphoreOne = new SemaphoreSlim(1);
await SimpleStatic.AddAsync(semaphoreOne);
Console.WriteLine(SimpleStatic.Value);
```

Класс, предоставляющий возможность асинхронной синхронизации с семантикой чтения/записи называется AsyncLock и есть в библиотеке Kanadeiar.Core.

Пример использования:

```csharp
static class SimpleStatic
{
    private static int _value;
    public static int Value => _value;
    public static async Task AddAsync(AsyncLock asyncLock)
    {
        await asyncLock.WaitAsync(LockMode.Shared); // Запросить общий доступ
        Console.WriteLine(_value); // Чтение из ресурса...
        asyncLock.Release(); // Снятие блокировки, чтобы ресурс стал доступен другим потокам
    }
}
// Использование в главном методе
await SimpleStatic.AddAsync(asyncLock);
```


