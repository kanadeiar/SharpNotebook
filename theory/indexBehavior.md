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

## Паттерн посредник

```csharp
interface IMediator
{
    void Notify(object sender, string ev);
}
class ConcreteMediator : IMediator
{
    private readonly Component1 _component1;
    /// **** другие компоненты
    public ConcreteMediator(Component1 component1)
    {
        _component1 = component1;
        _component1.SetMediator(this);
    }
    public void Notify(object sender, string ev)
    {
        if (ev == "B")
        {
            Console.WriteLine("Mediator reacts on A and triggers folowing operations:");
            _component1.DoB();
        }
        // ********** вызов других компонентов
    }
}
class BaseComponent
{
    protected IMediator _mediator;

    public BaseComponent(IMediator mediator = null)
    {
        _mediator = mediator;
    }

    public void SetMediator(IMediator mediator)
    {
        _mediator = mediator;
    }
}
class Component1 : BaseComponent
{
    public void DoA()
    {
        Console.WriteLine("Component 1 does A.");

        _mediator.Notify(this, "B");
    }
    public void DoB()
    {
        Console.WriteLine("Component 1 does B.");
    }
}
```
Использование:
```csharp
Component1 component1 = new Component1();
new ConcreteMediator(component1);
Console.WriteLine("Client triggets operation A.");
component1.DoA();
Console.WriteLine();
Console.ReadKey();
```

## Паттерн хранитель

Снимок — это поведенческий паттерн, позволяющий делать снимки внутреннего состояния объектов, а затем восстанавливать их.

При этом Снимок не раскрывает подробностей реализации объектов, и клиент не имеет доступа к защищённой информации объекта.

```csharp
class Originator
{
    private string _state;
    public Originator(string state)
    {
        _state = state;
        Console.WriteLine($"Originator: my initial state is {state}");
    }
    public void DoSomething()
    {
        Console.WriteLine("Originator: I'm doing something important.");
        _state = GenerateRandomString();
        Console.WriteLine($"Originator: and my state has changed to: {_state}");
    }
    private string GenerateRandomString(int length = 10)
    {
        string allowedSymbols = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ";
        string result = string.Empty;
        while (length > 0)
        {
            result += allowedSymbols[new Random().Next(0, allowedSymbols.Length)];
            Thread.Sleep(12);
            length--;
        }
        return result;
    }
    public IMemento Save()
    {
        return new ConcreteMemento(_state);
    }
    public void Restore(IMemento memento)
    {
        if (!(memento is ConcreteMemento))
        {
            throw new Exception("Unknown memento class " + memento.ToString());
        }
        _state = memento.GetState();
        Console.Write($"Originator: My state has changed to: {_state}");
    }
}
public interface IMemento
{
    string GetName();
    string GetState();
    DateTime GetDate();
}
class ConcreteMemento : IMemento
{
    private string _state;
    private DateTime _date;
    public ConcreteMemento(string state)
    {
        _state = state;
        _date = DateTime.Now;
    }
    public string GetState()
    {
        return _state;
    }
    public string GetName()
    {
        return $"{_date} / ({_state.Substring(0, 10)})...";
    }
    public DateTime GetDate()
    {
        return _date;
    }
}
class Caretaker
{
    private List<IMemento> _mementos = new List<IMemento>();
    private Originator _originator = null;
    public Caretaker(Originator originator)
    {
        _originator = originator;
    }
    public void Backup()
    {
        Console.WriteLine("\nCaretaker: Saving Originator's state...");
        _mementos.Add(_originator.Save());
    }
    public void Undo()
    {
        if (_mementos.Count == 0)
        {
            return;
        }
        var memento = _mementos.Last();
        _mementos.Remove(memento);
        Console.WriteLine("\nCaretaker: Restoring state to: " + memento.GetName());
        try
        {
            _originator.Restore(memento);
        }
        catch (Exception)
        {
            Undo();
        }
    }
    public void ShowHistory()
    {
        Console.WriteLine("Caretaker: Here's the list of mementos:");
        foreach (var memento in _mementos)
        {
            Console.WriteLine(memento.GetName());
        }
    }
}
```

Использование:
```csharp
var originator = new Originator("Super-duper-super-puper-super.");
var caretaker = new Caretaker(originator);
caretaker.Backup();
originator.DoSomething();
caretaker.Backup();
originator.DoSomething();
Console.WriteLine();
caretaker.ShowHistory();
caretaker.Undo();
caretaker.Undo();
Console.WriteLine();
Console.ReadKey();
```

## Паттерн наблюдатель

Наблюдатель — это поведенческий паттерн, который позволяет объектам оповещать другие объекты об изменениях своего состояния.

При этом наблюдатели могут свободно подписываться и отписываться от этих оповещений.

```csharp
public interface IObserver
{
    void Update(ISubject subject);
}
public interface ISubject
{
    void Attach(IObserver observer);
    void Detach(IObserver observer);
    void Notify();
}
public class Subject : ISubject
{
    public int State { get; set; } = 0;
    private List<IObserver> _observers = new List<IObserver>();
    public void Attach(IObserver observer)
    {
        Console.WriteLine("Attached");
        _observers.Add(observer);
    }
    public void Detach(IObserver observer)
    {
        _observers.Remove(observer);
        Console.WriteLine("Detached");
    }
    public void Notify()
    {
        Console.WriteLine("Subject - Notify");
        foreach (IObserver observer in _observers)
            observer.Update(this);
    }
    public void SomeBisinessLogic()
    {
        State++;
        Notify();
    }
}
class ConcreteObserverA : IObserver
{
    public void Update(ISubject subject)
    {
        if ((subject as Subject).State == 0 || (subject as Subject).State > 1)
        {
            Console.WriteLine("ConcreteObserverA: reacted to the event.");
        }
    }
}
```
Использование:
```csharp
var subject = new Subject();
var observerA = new ConcreteObserverA();
subject.Attach(observerA);
subject.SomeBisinessLogic();
subject.SomeBisinessLogic();
subject.Detach(observerA);
Console.WriteLine();
Console.ReadKey();
```

## Паттерн состояние

Состояние — это поведенческий паттерн, позволяющий динамически изменять поведение объекта при смене его состояния.

Поведения, зависящие от состояния, переезжают в отдельные классы. Первоначальный класс хранит ссылку на один из таких объектов-состояний и делегирует ему работу.

```csharp
class Context
{
    private State _state = null;
    public Context(State state)
    {
        TransitionTo(state);
    }
    public void TransitionTo(State state)
    {
        Console.WriteLine("Transition to {0}", state.GetType().Name);
        _state = state;
        _state.SetContext(this);
    }
    public void Request1()
    {
        _state.Handle1();
    }
}
abstract class State
{
    protected Context _context;
    public void SetContext(Context context)
    {
        _context = context;
    }
    public abstract void Handle1();
}
class ConcreteStateA : State
{
    public override void Handle1()
    {
        Console.WriteLine("ConcreteA change context");
        _context.TransitionTo(new ConcreteStateB());
    }
}
class ConcreteStateB : State
{
    public override void Handle1()
    {
        Console.WriteLine("ConcreteB handles request 1");
    }
}
```
Использование:
```csharp
var context = new Context(new ConcreteStateA());
context.Request1();
context.Request1();
Console.WriteLine();
Console.ReadKey();
```

## Паттерн стратегия

Стратегия — это поведенческий паттерн, выносит набор алгоритмов в собственные классы и делает их взаимозаменимыми.

Другие объекты содержат ссылку на объект-стратегию и делегируют ей работу. Программа может подменить этот объект другим, если требуется иной способ решения задачи.

```csharp
class Context
{
    private IStrategy _strategy;
    public Context()
    { }
    public Context(IStrategy strategy)
    {
        _strategy = strategy;
    }
    public void SetStrategy(IStrategy strategy)
    {
        _strategy = strategy;
    }
    public void DoSomeBusinessLogic()
    {
        Console.WriteLine("Context: Sorting data using the strategy");
        var results = _strategy.DoIt(new List<int> { 1, 2, 3 });
        var strs = string.Join(", ", results as List<int>);
        Console.WriteLine(strs);
    }
}
interface IStrategy
{
    object DoIt(object data);
}
class ConcreteStrategyA : IStrategy
{
    public object DoIt(object data)
    {
        var list = data as List<int>;
        list.Sort();
        return list;
    }
}
class ConcreteStrategyB : IStrategy
{
    public object DoIt(object data)
    {
        var list = data as List<int>;
        list.Sort();
        list.Reverse();
        return list;
    }
}
```
Использование:
```csharp
var context = new Strategy.Context();
context.SetStrategy(new Strategy.ConcreteStrategyA());
context.DoSomeBusinessLogic();
context.SetStrategy(new Strategy.ConcreteStrategyB());
context.DoSomeBusinessLogic();
Console.WriteLine();
Console.ReadKey();
```

## Паттерн шаблонный метод

Шаблонный метод — это поведенческий паттерн, задающий скелет алгоритма в суперклассе и заставляющий подклассы реализовать конкретные шаги этого алгоритма.

```csharp
abstract class AbstractClass
{
    public void TemplateMethod()
    {
        BaseOperation();
        RequiredOperation();
        Hook();
        BaseOperation();
    }
    protected void BaseOperation()
    {
        Console.WriteLine("Base operation");
    }
    protected abstract void RequiredOperation();
    protected virtual void Hook() { }
}
class ConcreteClass1 : AbstractClass
{
    protected override void RequiredOperation()
    {
        Console.WriteLine("Concrete class 1 implement operation");
    }
}
class ConcreteClass2 : AbstractClass
{
    protected override void RequiredOperation()
    {
        Console.WriteLine("Concrete class 2 implement operation");
    }
    protected override void Hook()
    {
        Console.WriteLine("Override hook");
    }
}
```
Использование:
```csharp
AbstractClass concrete1 = new ConcreteClass1();
concrete1.TemplateMethod();
AbstractClass concrete2 = new ConcreteClass2();
concrete2.TemplateMethod();
Console.WriteLine();
Console.ReadKey();
```

## Паттерн посетитель

Посетитель — это поведенческий паттерн, который позволяет добавить новую операцию для целой иерархии классов, не изменяя код этих классов.

```csharp
public interface IComponent
{
    void Accept(IVisitor visitor);
}
public class ConcreteComponentA : IComponent
{
    public void Accept(IVisitor visitor)
    {
        visitor.VisitConcreteComponentA(this);
    }
    public string ExclusiveMethodOfConcreteComponentA()
    {
        return "A";
    }
}
public interface IVisitor
{
    void VisitConcreteComponentA(ConcreteComponentA element);
}
class ConcreteVisitor : IVisitor
{
    public void VisitConcreteComponentA(ConcreteComponentA element)
    {
        Console.WriteLine(element.ExclusiveMethodOfConcreteComponentA() + " + Concrete Visitor 1");
    }
}
```
Использование:
```csharp
IComponent component = new ConcreteComponentA();
Console.WriteLine("visit to the base visitor interface");
var visitor = new ConcreteVisitor();
component.Accept(visitor);
Console.WriteLine();
Console.ReadKey();
```