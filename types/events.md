# События

Открытие доступа к членам типа делегата нарушает инкапсуляцию, что не только затруднит сопровождение кода (и отладку), но также сделает приложение уязвимым в плане безопасности, нужно применять события.

События - члены типа, обеспечивающие уведомление одного объекта о некоторых ситуациях других классов, которые могут случится. Методы обратного вызова позволяют объекту получать уведомления, на которые он подписался.

Событие обеспечивают типу возможность:

- Регистрации своей заинтересованности в событии.

- Отмена регистрации своей заинтересованности в событии.

- Оповещение зарегистрированных методов о произошедшем событии.

Модель событий основана на делегатах. Делегаты дают возможность использовать механизм функций обратного вызова, безопасного по отношению к типам.

При возникновении события объект, в котором оно возникло, должен передать дополнительную информацию объектам-получателям уведомления о событии. 

В результате обработки компилятором ключевого слова event вы автоматически получаете методы регистрации и отмены регистрации, а также все необходимые переменные-члены для типов делегатов. Такие переменные-члены с типами делегатов всегда объявляются как закрытые и потому они не доступны напрямую из объекта, инициирующего событие. В итоге ключевое слово event может использоваться для упрощения отправки специальным классом уведомлений внешним объектам.

## Создание и использование события

### На первом этапе создатся класс передачи информации

Создается специальный класс унаследованный от System.EventArgs, а имя типа должно заканчиватся на EventArgs. В нем определить свойства доступные только для чтения.

```csharp
internal class MyEventEventArgs : EventArgs
{
    private readonly int _value;
    public MyEventEventArgs(int value)
    {
        _value = value;
    }
    public int Value => _value;
}
```

>Если не нужно передавать какие-либо данные, можно использовать свойство EventArgs.Empty.

### На втором этапе определяется член-событие

Событие объявляется ключевым словом event, назначается область дейтвия, тип делегата, указывающий на прототип вызываемого метода и имя. Получатели уведомления о событии должны предоставлять метод обратного вызова, прототип которого соответствует делегату EventHandler<*>. 

Пример определения члена-события:

```csharp
internal class MySender
{
    public event EventHandler<ValueEventArgs>? NewValue;
}
```

Обобщенный делегат событий выглядит так:

```csharp
public delegate void EventHandler<TEventArgs> (Object s, TEventArgs e) where TEventArgs: EventArgs;
```

Прототип метода должен быть таким:

```csharp
void MyMethod(Object s, ValueEventArgs e);
```

Первым параметром метода должен быть тип Object для поддержки наследования и гибкости. Второй параметр должен называться "e". Все обработчики должны возвращать void.

### На тетьем этапе определяется метод, уведомляющий зарегистрированных объектов о событии

В классе должен быть виртуальный защищенный метод, вызываемый из кода класса и его потомков при возникновении события, в параметрах которого объект унаследованный от EventArgs. 

Определение такого метода уведомления:

```csharp
internal class MySender
{
    public event EventHandler<ValueEventArgs>? NewValue;
    protected virtual void OnNewValue(ValueEventArgs e)
    {
        if (Volatile.Read(ref NewValue) is { } temp)
        {
            temp(this, e);
        }
    }
}
```

В библиотеке Kanadeiar.Core расширяющий метод Kanadeiar.Core.Events.Extensions.Raise<TEArgs>() делает это красивее:

```csharp
e.Raise(this, ref NewValue);
```

Производный тип сможет свободно переопределять метод уведомления и изменять срабатывание события по своему усмотрению.

### На последнем четвертом этапе определяется метод, конвертирующий информацию на входе в нужное событие

Метод симулирует оповещение всех объектов о событии:

```csharp
internal class MySender
{
    ...
    public void Simulate(int value)
    {
        var e = new ValueEventArgs(value);
        OnNewValue(e);
    }
}
```

В результате уведомляются все заинтересованные объекты. 

А сам заинтересованный объект может вот так регистрироваться на уведомления этого класса:

```csharp
var sender = new MySender();
sender.NewValue += Method;
sender.Simulate(1);
// метод-цель
private static void Method(Object? s, ValueEventArgs e)
{
    System.Console.WriteLine($"New value = {e.Value}");
}
```

Пример заинтересованного класса:

```csharp
//объект, заинтересованный в уведомлении о событии
internal sealed class Fax
{
    public Fax(TestManager tm)
    {
        tm.Message += FaxMessage;
    }
    private void FaxMessage(object? sender, TestEventArgs e)
    {
        System.Console.WriteLine($"Fax message: {e.One} {e.Two}");
    }
    public void Unregister(TestManager tm)
    {
        tm.Message -= FaxMessage;
    }
}
```

Его использование:

```csharp
var manager = new TestManager();
var fax = new Fax(manager);
manager.SimulateNewMessage(1, 2);
fax.Unregister(manager);
```

Компилятор C# транслирует оператор += в код, регистрирующий объект для получения уведомления о событии:

```csharp
mm.add_NewValue(new EventHandler<ValueEventArgs>(this.Method));
```

Компилятор C# требует, чтобы для добавления и удаления делегатов из списка использовались операторы += и -=.

### Уровень компилятора

Строка кода определения события компилятором C# превращается в три конструкции:

- закрытое поле делегата, инициализированное null

- открытый метод add_xxx позволяющий регистрироватся для получения уведомлений о событии

- открытый метод remove_xxx позволяющий отменять регистрацию о получении уведомлений о событии

Когда получатель регистрируется для получения уведомления о событии, он добавляет в список экземпляр типа делегата. Отказ от регистрации реализуется удалением делегата.

Члены-события могут быть статическими или виртуальными. Среда CLR во время выполнения требует наличия методов доступа add и remove, сгенерированных компилятором C# на основе определения события.

## Явная регистрация событий

Компилятор C# позволяет разрботчикам явно реализовать события, управляя тем, как методы add & remove обращаются с делагатами обратных вызовов. Для эффективного хранения делегатов событий можно применить коллекцию, в которой идентификатор - это ключ, а список делегатов - значение. При регистрации события идентификатор события ищется в коллекции, если он найден, то новый делегат добавляется в список делегатов. Если идентификатор не найден, то он добавляется к делегатам.

При инициировании события идентификатор события ищется в коллекции. Если в коллекции нет элемента, то событие не регистрируется и делегат не вызывается. Но если находится - то вызывается делегат, связанный с идентификатором.

Пример реализации класса с коллекцией событий и списком делегатов:

```csharp
public sealed class EventSet
{
    private readonly Dictionary<EventKey, Delegate> _events = new Dictionary<EventKey, Delegate>();
    public void Add(EventKey key, Delegate handler)
    {
        Monitor.Enter(_events);
        _events.TryGetValue(key, out var d);
        _events[key] = Delegate.Combine(d, handler);
        Monitor.Exit(_events);
    }
    public void Remove(EventKey key, Delegate handler)
    {
        Monitor.Enter(_events);
        if (_events.TryGetValue(key, out var d))
        {
            d = Delegate.Remove(d, handler);
            if (d != null)
            {
                _events[key] = d;
            }
            else
            {
                _events.Remove(key);
            }
        }
        Monitor.Exit(_events);
    }
    public void Raise(EventKey key, Object sender, EventArgs e)
    {
        Monitor.Enter(_events);
        _events.TryGetValue(key, out var d);
        Monitor.Exit(_events);
        if (d != null)
        {
            d.DynamicInvoke(new Object[] { sender, e });
        }
    }
}
```

Использование этого класса в классе с событием:

```csharp
public class InvokeEventArgs : EventArgs { }

public class InvokeEvents
{
    private readonly EventSet _eventSet = new EventSet();
    protected EventSet EventSet { get => _eventSet; }
    // Идентификатор события
    protected static readonly EventKey _testEventKey = new EventKey();
    // Событие
    public event EventHandler<InvokeEventArgs> Test
    {
        add => _eventSet.Add(_testEventKey, value);
        remove => _eventSet.Remove(_testEventKey, value);
    }
    protected virtual void OnTest(InvokeEventArgs e)
    {
        _eventSet.Raise(_testEventKey, this, e);
    }
    public void Simulate()
    {
        OnTest(new InvokeEventArgs());
    }
}
```

Использование:

```csharp
var test = new InvokeEvents();
test.Test += new EventHandler(Method);
test.Simulate();
private static void Method(Object? s, InvokeEventArgs e)
{
    System.Console.WriteLine($"Test done");
}
```
