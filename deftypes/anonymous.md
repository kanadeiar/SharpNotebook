# Анонимные

## Анонимные типы

Механизм анонимных типов в C# позволяет автоматически объявить новый тип без названия, содержащих набор свойств, каким-либо образом связанных друг с другом. Значения в анонимных типах неизменны. Экземпляры анонимного типа не должны выходить за пределы метода. В прототипе метода не может содержаться параметр анонимного типа, так как задать анонимный тип невозможно. Метод не может возвращать ссылку на анонимный тип.

Пример определения нового анонимного объекта и класса и вывод его свойств в консоль:

```csharp
var test = new { One = 1, Two = 2 };
Console.WriteLine($"One = {test.One} Two = {test.Two}");
```

Компилятор C# определяет тип каждого выражения, создает закрытые поля этих типов, для каждого поля создает открытые поля доступа, переопределяет методы Equals, GetHashCode, ToString объекта и создает код этих методов.

Компилятор C# может выводить имена на основании имен переменных.

```csharp
var dt = DateTime.Now;
var test = new { dt.Year };
Console.WriteLine($"Year = {test.Year}");
```

Если компилятор видит определение множества анонимных типов с одинаковой структурой, то он создает одно определение для анонимного типа и множество экземпляров этого типа.

Анонимные типы поддерживают создание из них массивов.

```csharp
var arr = new[] 
{
    new { Value = 1 },
    new { Value = 2 },
    new { Value = 3 },
};
foreach (var item in arr)
{
    Console.WriteLine($"{item.Value}");
};
```

### Кортежи

Кортежи предназначены для того, чтоб быть легковесным механизмом передачи данных.

Относительно кортежей важно отметить два момента:

- поля не подвергаются проверке достоверности;

- определять собственные методы нельзя.

Определено несколько обобщенных кортежных типов унаследованных от System.Object, которые отличаются количеством обобщенных параметров. 

```csharp
[Serializable]
public class Tuple<T1> 
{
    private T1 m_Item1;
    public Tuple(T1 item1) { m_Item1 = item1; }
    public T1 Item { get { return m_Item1; } }
}
```

Значения в кортежах неизменны. Позволяют использовать CompareTo, Equals, GetHashCode, ToString, Size. 

Пример использования:

```csharp
private static Tuple<int, int> Swap(int a, int b)
{
    return new Tuple<int, int>(b, a);
}
// использование
var vals = Swap(1, 2);
System.Console.WriteLine($"{vals.Item1} {vals.Item2}");
```

В кортежах можно именовать свойства.

```csharp
(string FirstLetter, int TheNumber, string SecondLetter) valuesWithNames = ("a", 5, "c");
var valuesWithNames2 = (FirstLetter: "a", TheNumber: 5, SecondLetter: "c");
```

Cпециальные имена полей существуют только на этапе компиляции и не доступны при инспектировании кортежа во время выполнения с использованием рефлексии.

В C# имеется возможность выводить имена переменных в кортежан на основе передаваемых данных в кортеж изначально.

В кортежах имеется свойство эквивалентности (==) и неэквивалентности (!=) между собой. При проверке на неэквивалентность операции сравнения будут выполнять неявные преобразования типов данных внутри кортежей, включая сравнение допускающих и не допускающих null кортежей и/или свойств.

### Деконструирование кортежей

Деконструирование является термином, описывающим отделение свойств кортежа друг от друга с целью применения по одному. Деконструировать можно классы и структуры.

Пример определения деконструктора:

```csharp
struct Point
{
    public int X { get; set; }
    public int Y { get; set; }
    public (int XPos, int YPos) Deconstruct() => (X, Y);
}
```

Пример использования в обычном методе:

```csharp
var p1 = new Point { X = 1, Y = 2 };
var values = p1.Deconstruct();
Console.WriteLine($"X is: {values.XPos}");
Console.WriteLine($"Y is: {values.YPos}");
```

Пример использования деконструктора в выражении switch:

```csharp
var rez = p1.Deconstruct() switch
{
    (0, 0) => "начало",
    var (х, у) when х > 0 && у > 0 => "один",
    var (х, у) when х < 0 && у > 0 => "два",
    (_, _) => "отсутствие",
};
```

Можно упростить деконструктор, если установить в параметрах его модификаторы out:

```csharp
public void Deconstruct(out int XPos, out int YPos) => (XPos, YPos) = (X, Y);
// измененный пример:
var rez = p1 switch
{
...
};
```

## Анонимный метод

В языке C# можно использовать анонимные методы:

```csharp
var sender = new MySender();
sender.NewValue += delegate (object? s, ValueEventArgs e)
{
    System.Console.WriteLine($"New value = {e.Value}");
};
sender.Simulate(1);
```

Анонимные методы способны обращаться к локальным переменным кода, где они опеределены.

Анонимные методы также могут быть помечены как статические с целью предохранения инкапсуляции и гарантирования того, что они не привнесут какие-либо побочные эффекты в код, где они содержатся. Можно применять отбрасывание при использовании анонимных методов.

```csharp
var value = 10;
var simple = new Simple();
simple.Sample1 += static delegate(object? _, SimpleEventArgs e) 
{
    //value = 20; //теперь нельзя так делать
    Console.WriteLine("Вызов: " + e?.Message);
};
simple.Simulate();
```

## Лямбды

Лямбда-выражения представляют упрощенную запись анонимных методов. Лямбда-выражения позволяют создать емкие лаконичные методы, которые могут возвращать некоторое значение и которые можно передать в качестве параметров в другие методы.

Ламбда-выражения имеют следующий синтаксис: слева от лямбда-оператора => определяется список параметров, а справа блок выражений, использующий эти параметры.

```csharp	
(список_параметров) => выражение
```

Пример использования:

```csharp	
var hello = () => Console.WriteLine("Hello");
hello();       // Hello
hello();       // Hello
```

Если лямбда-выражение содержит несколько действий, то они помещаются в фигурные скобки:

```csharp	
var hello = () =>
{
    Console.Write("Hello ");
    Console.WriteLine("World");
};
hello();       // Hello World
```

Лямбда-выражение может возвращать результат. Возвращаемый результат можно указать после лямбда-оператора

```csharp	
var sum = (int x, int y) => x + y;
var sumResult = sum(4, 5);                  // 9
Console.WriteLine(sumResult);               // 9
var multiply = (x, y) => x * y;
var multiplyResult = multiply(4, 5);        // 20
Console.WriteLine(multiplyResult);          // 20

delegate int Operation(int x, int y);
```

### Лямбда-выражение как параметр метода

Лямбда-выражения можно передавать параметрам метода, которые представляют делегат.

Пример:

```csharp	
int[] integers = { 1, 2, 3, 4, 5, 6, 7, 8, 9 };
 
// найдем сумму чисел больше 5
int result1 = Sum(integers, x => x > 5);
Console.WriteLine(result1); // 30
 
// найдем сумму четных чисел
int result2 = Sum(integers, x => x % 2 == 0);
Console.WriteLine(result2);  //20
 
int Sum(int[] numbers, IsEqual func)
{
    int result = 0;
    foreach (int i in numbers)
    {
        if (func(i))
            result += i;
    }
    return result;
}
 
delegate bool IsEqual(int x);
```

### Лямбда-выражение как результат метода

Метод также может возвращать лямбда-выражение. В этом случае возвращаемым типом метода выступает делегат, которому соответствует возвращаемое лямбда-выражение.

Пример:

```csharp	
var operation = SelectOperation(OperationType.Add);
Console.WriteLine(operation(10, 4));    // 14 
operation = SelectOperation(OperationType.Subtract);
Console.WriteLine(operation(10, 4));    // 6 
operation = SelectOperation(OperationType.Multiply);
Console.WriteLine(operation(10, 4));    // 40

Operation SelectOperation(OperationType opType)
{
    switch (opType)
    {
        case OperationType.Add: return (x, y) => x + y;
        case OperationType.Subtract: return (x, y) => x - y;
        default: return (x, y) => x * y;
    }
}
enum OperationType
{
    Add, Subtract, Multiply
}
delegate int Operation(int x, int y);
```





