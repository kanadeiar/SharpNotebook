# Поведения паттерны

## Паттерн цепочка обязанностей

Цепочка обязанностей — это поведенческий паттерн, позволяющий передавать запрос по цепочке потенциальных обработчиков, пока один из них не обработает запрос.

Избавляет от жёсткой привязки отправителя запроса к его получателю, позволяя выстраивать цепь из различных обработчиков динамически.

```csharp
interface IHandler
{
    IHandler SetNext(IHandler handler);
    object Handle(object request);
}
abstract class AbstractHandler : IHandler
{
    private IHandler _nextHandler;
    public IHandler SetNext(IHandler handler)
    {
        _nextHandler = handler;
        return handler;
    }
    public virtual object Handle(object request)
    {
        if (_nextHandler != null)
        {
            return _nextHandler.Handle(request);
        }
        else
        {
            return null;
        }
    }
}
class MonkeyHandler : AbstractHandler
{
    public override object Handle(object request)
    {
        if ((request as string) == "Banana")
        {
            return $"Monkey: eat the {request}";
        }
        else
        {
            return base.Handle(request);
        }
    }
}
class DogHandler : AbstractHandler
{
    public override object Handle(object request)
    {
        if ((request as string) == "Meat")
        {
            return $"Dog: eat the {request}";
        }
        else
        {
            return base.Handle(request);
        }
    }
}
```

Использование:
```csharp
Console.WriteLine("Chain");
var monkey = new MonkeyHandler();
var dog = new DogHandler();
monkey.SetNext(dog);
Console.WriteLine("Chain: Monkey -> Dog");
Console.WriteLine(monkey.Handle("Meat"));
Console.WriteLine("Chain: Monkey");
Console.WriteLine(monkey.Handle("Banana"));
Console.WriteLine();
Console.ReadKey();
```

## Паттерн команда

Команда — это поведенческий паттерн, позволяющий заворачивать запросы или простые операции в отдельные объекты.

Это позволяет откладывать выполнение команд, выстраивать их в очереди, а также хранить историю и делать отмену.

```csharp
interface ICommand
{
    void Execute();
}

class SimpleCommand : ICommand
{
    private readonly string _payload;
    public SimpleCommand(string payload)
    {
        _payload = payload;
    }
    public void Execute()
    {
        Console.WriteLine($"SimpleCommand: See {_payload}");
    }
}

class Invoker
{
    private ICommand _onStart;
    private ICommand _onStop;
    public void SetOnStart(ICommand command)
    {
        _onStart = command;
    }
    public void SetOnStop(ICommand command)
    {
        _onStop = command;
    }
    public void Invoke()
    {
        if (_onStart is ICommand)
            _onStart.Execute();
        if (_onStop is ICommand)
            _onStop.Execute();
    }
}
```

Использование:
```csharp
var invoker = new Invoker();
invoker.SetOnStart(new SimpleCommand("Say Hi!"));
invoker.SetOnStop(new SimpleCommand("Say Bye!"));
invoker.Invoke();
Console.WriteLine();
Console.ReadKey();
```

## Паттерн итератор

Итератор — это поведенческий паттерн, позволяющий последовательно обходить сложную коллекцию, без раскрытия деталей её реализации.

Благодаря Итератору, клиент может обходить разные коллекции одним и тем же способом, используя единый интерфейс итераторов.

```csharp
abstract class Iterator : IEnumerator
{
    object IEnumerator.Current => Current();
    public abstract int Key();
    public abstract object Current();
    public abstract bool MoveNext();
    public abstract void Reset();
}
abstract class IteratorAggregate : IEnumerable
{
    public abstract IEnumerator GetEnumerator();
}
class AlphabeticalOrderIterator : Iterator
{
    private WordsCollection _collection;
    private int _position = -1;
    private bool _reverse = false;
    public AlphabeticalOrderIterator(WordsCollection collection, bool reverse = false)
    {
        _collection = collection;
        _reverse = reverse;
        if (reverse)
        {
            _position = collection.getItems().Count;
        }
    }
    public override object Current()
    {
        return _collection.getItems()[_position];
    }
    public override int Key()
    {
        return _position;
    }
    public override bool MoveNext()
    {
        int updatedPosition = _position + (_reverse ? -1 : 1);

        if (updatedPosition >= 0 && updatedPosition < _collection.getItems().Count)
        {
            _position = updatedPosition;
            return true;
        }
        else
        {
            return false;
        }
    }
    public override void Reset()
    {
        _position = _reverse ? _collection.getItems().Count - 1 : 0;
    }
}
class WordsCollection : IteratorAggregate
{
    List<string> _collection = new List<string>();
    bool _direction = false;
    public void ReverseDirection()
    {
        _direction = !_direction;
    }
    public List<string> getItems()
    {
        return _collection;
    }
    public void Add(string item)
    {
        _collection.Add(item);
    }
    public override IEnumerator GetEnumerator()
    {
        return new AlphabeticalOrderIterator(this, _direction);
    }
}
```

Использование:
```csharp
var collection = new WordsCollection();
collection.Add("First");
collection.Add("Second");
collection.Add("Third");
foreach (var element in collection)
{
    Console.WriteLine(element);
}
Console.WriteLine("\nReverse:");
collection.ReverseDirection();
foreach (var element in collection)
{
    Console.WriteLine(element);
}
Console.WriteLine();
Console.ReadKey();
```