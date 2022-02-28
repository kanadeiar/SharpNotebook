# Паттерн абстрактная фабрика

Абстрактная фабрика — это порождающий паттерн проектирования, который решает проблему создания целых семейств связанных продуктов, без указания конкретных классов продуктов.

Абстрактная фабрика задаёт интерфейс создания всех доступных типов продуктов, а каждая конкретная реализация фабрики порождает продукты одной из вариаций. Клиентский код вызывает методы фабрики для получения продуктов, вместо самостоятельного создания с помощью оператора new. При этом фабрика сама следит за тем, чтобы создать продукт нужной вариации.

```csharp
namespace ConsoleAppGaF.Creational;

public interface IAbstractFactory
{
    IAbstractProductA CreateProductA();
    IAbstractProductB CreateProductB();
}
class ConcreteFactory1 : IAbstractFactory
{
    public IAbstractProductA CreateProductA()
    {
        return new ConcreteProductA1();
    }
    public IAbstractProductB CreateProductB()
    {
        return new ConcreteProductB1();
    }
}
class ConcreteFactory2 : IAbstractFactory
{
    public IAbstractProductA CreateProductA()
    {
        return new ConcreteProductA2();
    }
    public IAbstractProductB CreateProductB()
    {
        return new ConcreteProductB2();
    }
}
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
class ConcreteProductA2 : IAbstractProductA
{
    public string UsefulFunctionA()
    {
        return "The result of the product A2.";
    }
}
public interface IAbstractProductB
{
    string UsefulFunctionB();
    string AnotherUsefulFunctionB(IAbstractProductA collaborator);
}
class ConcreteProductB1 : IAbstractProductB
{
    public string UsefulFunctionB()
    {
        return "The result of the product B1.";
    }
    public string AnotherUsefulFunctionB(IAbstractProductA collaborator)
    {
        var result = collaborator.UsefulFunctionA();

        return $"The result of the B1 collaborating with the ({result})";
    }
}
class ConcreteProductB2 : IAbstractProductB
{
    public string UsefulFunctionB()
    {
        return "The result of the product B2.";
    }
    public string AnotherUsefulFunctionB(IAbstractProductA collaborator)
    {
        var result = collaborator.UsefulFunctionA();

        return $"The result of the B2 collaborating with the ({result})";
    }
}
```
Использование:
```csharp
Console.WriteLine("GoF Abstract Fabric");

Console.WriteLine("Client: Testing client code with the first factory type...");
ClientMethod(new ConcreteFactory1());
Console.WriteLine();

Console.WriteLine("Client: Testing the same client code with the second factory type...");
ClientMethod(new ConcreteFactory2());

Console.ReadKey();

void ClientMethod(IAbstractFactory factory)
{
    var productA = factory.CreateProductA();
    var productB = factory.CreateProductB();

    Console.WriteLine(productB.UsefulFunctionB());
    Console.WriteLine(productB.AnotherUsefulFunctionB(productA));
}
```



