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
