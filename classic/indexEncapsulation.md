# Инкапсуляция (модификаторы доступа, свойства, автоматические свойства)


## Модификаторы доступа

В языке C# типы (классы, интерфейсы, структуры, перечисления и делегаты) и их члены (свойства, методы, конструкторы и поля) описываются с использованием модификаторов, устанавливающими видимостью элементов для других частей приложения.

Модификаторы доступа C#
```csharp
Модификатор         Применение                  Смысл
public              Типы или члены типов        Не имеют ограничений доступа, доступен из внешних сборок
private             Члены типов или вложенные   Могут быть доступны только классу, где определены
protected           Члены типов или вложенные   Могут использоватся классом, который их определяет и любым дочерним классам.
internal            Типы или члены типов        Доступны только в рамках текущей сборки.
protected internal  Члены типов или вложенные   Такой элемент будет доступен внутри сборки, внутри определяющего класса и для всех наследников
```
По умолчанию все типы в языке C# устанавливаются как internal, а члены типов являются закрытыми private. 

То есть этот код:
```csharp
class MyClass
{
    MyClass() {}
}
```
Равносилен этому:
```csharp
internal class MyClass
{
    private MyClass() {}
}
```
А для того чтобы тип и его члены были доступны везде нужно сделать так:
```csharp
public class MyClass
{
    public MyClass() {}
}
```

Модификаторы доступа могут применятся к вложенному типу. Вложенный тип - это тип, объявленный внутри области видимости класса или структуры.
```csharp
public class MyClass
{
    private class MiniClass { }
}
```

## Принципы инкапсуляции

Данные класса не должны быть доступны напрямую через его экземпляр, данные класса должны быть закрытыми. Если нужно получить доступ к данным, то нужно получать его косвенно, используя открытые члены.

Инкапсуляция предусматривает вмето открытых полей определение закрытых данных, управление которыми осуществляется с применением одного из двух:

- пары открытх методово доступа и изменения.

- открытого свойства.

Хорошо инкапсулированный класс должен защищать свои данные и скрывать подробности своего функционирования от внешнего мира.

>Члены класса, которые определяют состояние объекта, не должны быть публичными, однако нормально иметь открытые константы и открытые поля, допускающе только чтение.

Пример традиционного подхода:
```csharp
class Sample
{
    private string name;
    public string GetName()
    {
        return name;
    }
    public void SetName(string name)
    {
        this.name = name;
    }
}
```
Использование:
```csharp
Sample sample = new Sample();
sample.SetName("and");
string name = sample.GetName();
```
# Свойства

В языке C# лучше использовать свойства для доступа к закрытым полям. Свойства - упрощенное представление настоящих методов доступа и изменения. Пример класса со свойствами:
```csharp
class Sample
{
    private string name;
    public string Name
    {
        get { return name; }
        set { name = value; }
    }
}
```
Использование:
```csharp
Sample sample = new Sample();
sample.Name = "and";
string name = sample.Name;
```
Методы свойства get & set могут записываться в виде членов, сжатых до выражений. Пример:
```csharp
class Sample
{
    private string name;
    public string Name
    {
        get => name;
        set => name = value;
    }
}
```
Для получение и установки значения внутри класса нужно всегда применять свойства. Пример с конструктором:
```csharp
class Sample
{
    private string name;
    public string Name
    {
        get => name;
        set => name = value;
    }

    public Sample(string name)
    {
        Name = name;
    }
}
```
Могут быть свойства только для чтения и только для записи. Пример:
```csharp
class Sample
{
    private string name;
    public string Name => name;
    public string SetName
    {
        set => name = value;
    }
}
```
Может быть статическое свойство:
```csharp
class Sample
{
    private string name;
    public string Name => name;
    public string SetName
    {
        set => name = value;
    }
    private static int count = 1;
    public static int Count
    {
        get => count;
        set => count = value;
    }
}
```
Использование:
```csharp
Sample.Count = 10;
int count = Sample.Count;
```



## Автоматические свойства

Доступно вместо полного написания свойства и приватного поля данных использовать синтаксис автоматических свойств. Это свойство перекладывает работу по определению закрытых полей и связанных свойств на компилятор. Пример класса:
```csharp
class Sample
{
    public string Name { get; set; }
}
```
Можно определять автоматическое свойство только для чтения:
```csharp
class Sample
{
    public string Name { get; }
}
```
>Можно написать prop и два раза нажать таб - среда сгенерирует начальный код автоматического свойства.

Автоматическим свойствам присваиваются стандартные значения - булевской переменной - false, а для числовых данных - 0. Однако для переменной дригого класса - null.

Нужно, если требуется, устанавливать первоначальные значения в такие свойства. Пример:
```csharp
class Sample
{
    public string Name { get; set; }
    public int Age { get; set; }
    public Sample()
    {
        Name = "noname";
        Age = 18;
    }
}
```
Однако лучше присваивать начальные значения автоматическим свойствам прямо при объявлении этого свойства.
```csharp
class Sample
{
    public string Name { get; set; } = "default";
    public int Age { get; set; } = 18;
}
```

