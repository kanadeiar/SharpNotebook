# Структурные паттерны

## Паттерн адаптер

Адаптер — это структурный паттерн, который позволяет подружить несовместимые объекты.

Адаптер выступает прослойкой между двумя объектами, превращая вызовы одного в вызовы понятные другому.

```csharp
public sealed class Adaptee
{
    public string GetSpecificRequest()
    {
        return "Specific request.";
    }
}
public interface ITarget
{
    string GetMyRequest();
}
public class Adapter : ITarget
{
    private readonly Adaptee _adaptee;
    public Adapter(Adaptee adaptee)
    {
        _adaptee = adaptee;
    }
    public string GetMyRequest()
    {
        return $"This is adapter: {_adaptee.GetSpecificRequest()}";
    }
}
```

Использование:
```csharp
Console.WriteLine("Adapter");
var ad = new Adaptee();
var target = new Adapter(ad);
Console.WriteLine($"My adapter: {target.GetMyRequest()}");        
Console.ReadKey();
```

## Паттерн мост

Применимость: Паттерн Мост особенно полезен когда вам приходится делать кросс-платформенные приложения, поддерживать несколько типов баз данных или работать с разными поставщиками похожего API (например, cloud-сервисы, социальные сети и т. д.)

Признаки применения паттерна: Если в программе чётко выделены классы «управления» и несколько видов классов «платформ», причём управляющие объекты делегируют выполнение платформам, то можно сказать, что у вас используется Мост.

```csharp
/// <summary> Основа </summary>
public class Abstraction
{
    private readonly IImplementation _implementation;
    public Abstraction(IImplementation implementation)
    {
        _implementation = implementation;
    }
    public virtual string Operation()
    {
        return $"Abstract: base with {_implementation.OperationImplementation()}";
    }
}
/// <summary> Дополнение </summary>
public class ExtendedAbstaction : Abstraction
{
    public ExtendedAbstaction(IImplementation implementation) : base(implementation)
    {
    }
    public override string Operation()
    {
        return $"Extended Abstraction: {base.Operation()}";
    }
}

public interface IImplementation
{
    string OperationImplementation();
}
// Одна из платформ, реализация на эту платформу
public sealed class ConcreteImplementationA : IImplementation
{
    public string OperationImplementation()
    {
        return "ConcreteImplementationA: The result in platform A.\n";
    }
}
```

Использование:
```csharp
Console.WriteLine("Bridge");
var abstraction = new Abstraction(new ConcreteImplementationA());
Console.Write(abstraction.Operation());
var abstractionExt = new ExtendedAbstaction(new ConcreteImplementationA());
Console.Write(abstractionExt.Operation());
Console.WriteLine();
Console.ReadKey();
```

## Паттерн компоновщик

Компоновщик — это структурный паттерн, который позволяет создавать дерево объектов и работать с ним так же, как и с единичным объектом.

Компоновщик давно стал синонимом всех задач, связанных с построением дерева объектов. Все операции компоновщика основаны на рекурсии и «суммировании» результатов на ветвях дерева.

```csharp
interface IComponent
{
    public string Operation();
    public virtual void Add(IComponent component) => throw new NotImplementedException();
    public virtual void Remove(IComponent component) => throw new NotImplementedException();
    public virtual bool IsComposite() => true;
}
class Leaf : IComponent
{
    public string Operation()
    {
        return "Leaf";
    }
    public bool IsComposite()
    {
        return false;
    }
}
class Composite : IComponent
{
    protected List<IComponent> _children = new List<IComponent>();
    public void Add(IComponent component)
    {
        _children.Add(component);
    }
    public void Remove(IComponent component)
    {
        _children.Remove(component);
    }
    public string Operation()
    {
        int i = 0;
        string result = "Branch( ";
        foreach (var component in _children)
        {
            result += component.Operation();
            if (i != _children.Count - 1)
            {
                result += " + ";
            }
            i++;
        }
        return result + " )";
    }
}
```

Использование:
```csharp
Console.WriteLine("Composite");
Leaf leaf = new Leaf();
Console.WriteLine($"RESULT: {leaf.Operation()}\n");
Composite tree = new Composite();
Composite branch1 = new Composite();
branch1.Add(new Leaf());
branch1.Add(new Leaf());
Composite branch2 = new Composite();
branch2.Add(new Leaf());
tree.Add(branch1);
tree.Add(branch2);
Console.WriteLine($"RESULT: {tree.Operation()}\n");
Console.WriteLine();
Console.ReadKey();
```