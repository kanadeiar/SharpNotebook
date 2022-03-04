# Порождающие паттерны

## Паттерн абстрактная фабрика

Абстрактная фабрика — это порождающий паттерн проектирования, который решает проблему создания целых семейств связанных продуктов, без указания конкретных классов продуктов.

Абстрактная фабрика задаёт интерфейс создания всех доступных типов продуктов, а каждая конкретная реализация фабрики порождает продукты одной из вариаций. Клиентский код вызывает методы фабрики для получения продуктов, вместо самостоятельного создания с помощью оператора new. При этом фабрика сама следит за тем, чтобы создать продукт нужной вариации.

```csharp
//абстрактная фабрика
public interface IAbstractFactory
{
    IAbstractProductA CreateProductA();
}
class ConcreteFactory1 : IAbstractFactory
{
    public IAbstractProductA CreateProductA()
    {
        return new ConcreteProductA1();
    }
}
//абстрактный продукт
public interface IAbstractProductA
{
    string UsefulFunctionA();
}
class ConcreteProductA1 : IAbstractProductA
{
    public string UsefulFunctionA()
    {
        return "The result of the product A1.";
    }
}
```

Использование:
```csharp
Console.WriteLine("Client: Testing client code with the factory type...");
ClientMethod(new ConcreteFactory1());
Console.WriteLine();

Console.ReadKey();

void ClientMethod(IAbstractFactory factory)
{
    var productA = factory.CreateProductA();
    Console.WriteLine(productA.UsefulFunctionA());
}
```

## Паттерн строитель

Строитель — это порождающий паттерн проектирования, который позволяет создавать объекты пошагово.

В отличие от других порождающих паттернов, Строитель позволяет производить различные продукты, используя один и тот же процесс строительства.

```csharp
//абстрактный строитель
public interface IBuilder
{
    IBuilder BuildPartA();
    IBuilder BuildPartB();
}
public class ConcreteBuilder : IBuilder
{
    private Product _product = new Product();
    public void Reset()
    {
        _product = new Product();
    }
    public IBuilder BuildPartA()
    {
        _product.Add("PartA");
        return this;
    }
    public IBuilder BuildPartB()
    {
        _product.Add("PartB");
        return this;
    }
    public Product Get()
    {
        var result = _product;
        Reset();
        return result;
    }
}
public class Product
{
    public List<object> Parts = new List<object>();
    public void Add(string p) => Parts.Add(p);
}
//директор - выполнение по шагам
public class Direcrtor
{
    private IBuilder _builder;
    public IBuilder Builder
    {
        set => _builder = value;
    }
    public void BuildMinimal() => _builder.BuildPartA();
    public void BuildMaximum()
    {
        _builder.BuildPartA().BuildPartB();
    }
}
```

Использование:
```csharp
Console.WriteLine("Testing builder");
var director = new Direcrtor();
var builder = new ConcreteBuilder();
director.Builder = builder;

Console.WriteLine("Basic");
director.BuildMinimal();
foreach(var e in builder.Get().Parts)
    Console.WriteLine("- " + e);
Console.WriteLine("Maximum");
director.BuildMaximum();
foreach (var e in builder.Get().Parts)
    Console.WriteLine("- " + e);
Console.WriteLine();
Console.ReadKey();
```

## Фабричный метод

Фабричный метод — это порождающий паттерн проектирования, который решает проблему создания различных продуктов, без указания конкретных классов продуктов.

Фабричный метод задаёт метод, который следует использовать вместо вызова оператора new для создания объектов-продуктов. Подклассы могут переопределить этот метод, чтобы изменять тип создаваемых продуктов.

```csharp
/// <summary> Абстрактный создатель </summary>
interface ICreator
{
    public IProduct FactoryMethod();
    public string SomeOperation()
    {
        var product = FactoryMethod();
        var result = $"Creator: {product.Operation()}";
        return result;
    }
}
class ConcreteCreator1 : ICreator
{
    public IProduct FactoryMethod()
    {
        return new ConcreteProduct1();
    }
}
/// <summary> Продукт </summary>
public interface IProduct
{
    string Operation();
}
class ConcreteProduct1 : IProduct
{
    public string Operation()
    {
        return "[Result of ConcreteProduct1]";
    }
}
```

Использование:
```csharp
Console.WriteLine("GoF Builder");
Console.WriteLine("Testing FacrotyMethods");
ClientCode(new ConcreteCreator1());
void ClientCode(ICreator creator)
{
    Console.WriteLine($"Client: {creator.SomeOperation()}");
}
Console.WriteLine();
Console.ReadKey();
```

## Прототип

Прототип — это порождающий паттерн, который позволяет копировать объекты любой сложности без привязки к их конкретным классам.

Все классы—Прототипы имеют общий интерфейс. Поэтому вы можете копировать объекты, не обращая внимания на их конкретные типы и всегда быть уверены, что получите точную копию. Клонирование совершается самим объектом-прототипом, что позволяет ему скопировать значения всех полей, даже приватных.

```csharp
public class Person
{
    public string Name { get; set; }
    public Person ShallowCopy()
    {
        return (Person)MemberwiseClone();
    }
    public Person Copy()
    {
        var clone = (Person)MemberwiseClone();
        clone.Name = Name;
        return clone;
    }
}
```
Использование:
```csharp
Console.WriteLine("GoF Builder");
Console.WriteLine("Testing Prototype");

var p1 = new Person { Name = "Test" };

var p2 = p1.ShallowCopy();
var p3 = p1.Copy();
```

## Одиночка

Одиночка — это порождающий паттерн проектирования, который гарантирует, что у класса есть только один экземпляр, и предоставляет к нему глобальную точку доступа.

```csharp
public sealed class Singleton
{
    private Singleton() { }
    private static Singleton _instance;
    public static Singleton GetInstance() => _instance ??= new Singleton();
}
```
Использование:
```csharp
var s1 = Singleton.GetInstance();
var s2 = Singleton.GetInstance();
if (s1 == s2)
{
    Console.WriteLine("Equals");
}
Console.WriteLine();
Console.ReadKey();
```