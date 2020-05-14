# Объекты (структуры, кортежи, перегрузка операций, сборка мусора)

## Структуры

В языке C# имеется составной тип данных, называется он структурой. Может в своем составе содержать обычные переменные и методы. Хорошо подходят для атомарных сущностей. Типы, которые могут содержать любое количество полей данных и членов, действующих на таких полях.
>Структуру можно считать легковесным типом классов, он поддерживает инкапсуляцию, но не может применяться для построения семейства взаимосвязанных классов. Когда нет потребности в наследовании, нужно применять структуры.

Структура:
```csharp
struct MyStruct
{
    public int a;
    MyStruct(int a) //конструктор
    {
        this.a = a;
    }
    public MyStruct Plus(MyStruct p) //метод
    {
        MyStruct y;
        y.a = a + p.a;
        return y;
    }
    public string ToString() //перегруженный метод получения строкового заначения
    {
        return $"<<a = {a}>>";
    }
}
```
Структуры не поддерживают один из основных принципов ООП - наследование. Назначение - объединить в себе небольшой объем данных.

Использование:
```csharp
MyStruct mys = MyStruct(1);
```

## Кортежи

Кортежи - легковесные структуры данных, содержащие множество полей. Кортежи используют тип данных ValueTurple, сберегая значительный объем памяти. Тип данных ValueTurple создает различные структуры на основе количества свойств для кортежа. Кроме этого каждому свойству кортежа можно назначать специфическое имя, что значительно повышает удобство работы с ним.
```csharp
(string, int) values1 = ("1", 4);
var values2 = ("1", 5);
```
>Для работы с кортежами понадобится установить NuGet пакет System.ValueTuple.

По умолчанию компилятор назначает каждому элементу кортежа имя ItemX.
```csharp
WriteLine(values1.Item1);
WriteLine(values1.Item2);
```
Каждому элементу кортежа можно назначить свое имя:
```csharp
(string Fisrt, int Second) values1 = ("1", 4);
var values2 = (First : "1", Second : 5);
```
Теперь можно обращаться как по имени так и способом по умолчанию.
```csharp
WriteLine(values1.Fisrt);
WriteLine(values1.Second);
WriteLine(values1.Item1);
WriteLine(values1.Item2);
```
>Специальные имена однако, существуют только на этапе компиляции, не доступны во время исполнения с использованием рефлексии.

В версии C# 7.1 появилась возможность выводить имена свойств кортежей при определенных обстоятельствах.
```csharp
var myCort = new {one = "super", two = 5};
var myTwoCort = (myCort.two, myCort.one);
WriteLine($"{myTwoCort.one} {myTwoCort.two}");

int count = 5;
string label = "Colors used in the map";
var pair = (count, label);
```

Кортежи удобно применять как вертаемое значение от методов:
```csharp
static (string a, int b) getTuple()
{
    return ("one", 5);
}
```
Вызов такого метода (второй вызов - деконструирование кортежа, с целью применения по одному):
```csharp
var myData = getTuple();
WriteLine(myData.a);
WriteLine(myData.b);
var (on, tw) = getTuple();
WriteLine(on);
WriteLine(tw);
```

Использование отбрасывания с кортежами - когда одна из переменных не нужна - нужно вставить пустым заполнителем _:
```csharp
var (one, _) = getTuple(); 
WriteLine($"{one}");
```

Кортежи применяются для деконструирования специальных типов. Пример такого типа:
```csharp
struct Super
{
    public int a;
    public bool b;
    public (int aC, bool bC) Deconstruct() => (a, b);
}
```
Применение:
```csharp
Super super = new Super {a = 10, b = true};
var (aVal, bVal) = super.Deconstruct();
WriteLine(aVal);
WriteLine(bVal);
```
Начиная с версии C# 7.3 типы кортежей поддерживают операторы == и !=.
```csharp
var left = (a: 5, b: 10);
var right = (a: 5, b: 10);
Console.WriteLine(left == right); // displays 'true'
```


## Объекты 

Класс - это будущий объект. В классе, как и в структуре, мы поисываем будущий объект. Для создания объекта нужно сначало объявить переменную - ссылку на объект, а потом попросить выделить место в памяти для объекта. Все объееты относятся к ссылочным типам данных. 

Объект создается в два этапа - сначала создается ссылка на объект, а затем идет инициализация, то есть создается объект в куче. За освобождение памяти можно не беспокоится - об этом позаботится автоматический сборщик мусора.

Объявление класса:
```csharp
class MyClass
{
}
```

Создание объекта класса MyClass:
```csharp
MyClass myObj = new MyClass();
```

## Статические и нестатические поля класса 

Статическое поле принадлежит классу, или всем объектам класса.
Нестатическое поле принадлежит только объекту класса, для доступа к нему нужно создавать объект.
```csharp
MyClass.stat_variable = 10;
MyClass myclass = new MyClass();
myclass.variable = 20;
Console.WriteLine($"{MyClass.stat_variable} {myclass.variable}"); // 10 20
ReadKey();
```


# Продвинутое

## Перегрузка операций

Язык C# дает возможность строить классы и структуры, которые могут униально реагировать на один и тот же набор базовых лексем - таких как сложение и вычитание. Вот список операций, которые могут быть перегружены:
```csharp
Операция C#                 Возможность
+ - ! ~ ++ -- true false    Можно
+ - * / % & | ^ << >>       Можно
== != < > <= >=             Можно, но нужно парно перегружать (> и <, == и !=)
```
Операции которые никак не могу быть перегружены:
```csharp
Операция C#                     Возможность
[]                              Не может быть. Ту же функциональность обеспечивает индексатор
()                              Не может быть. Ту же функциональность обеспечивают спец методы преобразования.
+= -= *= /= %= &= |= ^= <<= >>= Нельзя. Получение автоматически, когда перегружается бинарная операция.
```
## Перегрузка бинарных операций

Пример перегруженных операций класс:
```csharp
public class Number
{
    public int Val { get; set; }
    public static Number operator +(Number n1, Number n2) =>
        new Number { Val = n1.Val + n2.Val };
    public static Number operator -(Number n1, Number n2) =>
        new Number { Val = n1.Val - n2.Val };
}
```
Использование:
```csharp
var n1 = new Number { Val = 1 };
var n2 = new Number { Val = 10 };
var n3 = n1 + n2;
var n4 = n1 - n2;
```
В качестве второго параметра может выступать другой тип даных, в этом случае нужно определить обе версии метода.
Пример операции класс:
```csharp
public class Number
{
    public int Val { get; set; }
    public static Number operator +(Number n1, int val) =>
        new Number { Val = n1.Val + val };
    public static Number operator +(int val, Number n2) =>
        new Number { Val = val + n2.Val };
}
```
Использование:
```csharp
var n1 = new Number { Val = 1 };
Number n3 = n1 + 10;
Number n4 = 5 + n1;
```
>Операции += и -= уже перегружены автоматическии:
```csharp
n1 += n2;
```
## Перегрузка унарных операций
Пример класса:
```csharp
public class Number
{
    public int Val { get; set; }
    public static Number operator ++(Number n1) =>
        new Number { Val = n1.Val + 1 };
    public static Number operator --(Number n1) =>
        new Number { Val = n1.Val - 1 };
}
```
Использование:
```csharp
var n1 = new Number { Val = 1 };
var n2 = new Number { Val = 10 };
n1++;
++n2;
n2--;
--n1;
```
>В языке C# не допускается перегружать операции префиксного и посфикснго инкремента/декремента. Это автоматически обрабатывается корректно.

## Перегрузка операций эквивалентности
Пример класса:
```csharp
public class Number
{
    public int Val { get; set; }
    public override bool Equals(object obj) => obj.ToString() == this.ToString();
    public static bool operator ==(Number n1, Number n2) => n1.Equals(n2);
    public static bool operator !=(Number n1, Number n2) => !n1.Equals(n2);
}
```
Использование:
```csharp
var n1 = new Number { Val = 1 };
var n2 = new Number { Val = 10 };
bool b = n1 == n2;
bool b2 = n1 != n2;
```
## Перегрузка операций сравнения
Для перегрузок операций сравнения рекомендуется применять обобщенный эквивалент интерфейса IComparable. Пример класса с реализованным этим интерфейсом:
```csharp
public class Number : IComparable<Number>
{
    public int Val { get; set; }
    public int CompareTo(Number other)
    {
        if (this.Val > other.Val)
            return 1;
        if (this.Val < other.Val)
            return -1;
        return 0;
    }
    public static bool operator <(Number n1, Number n2) =>
        n1.CompareTo(n2) < 0;
    public static bool operator >(Number n1, Number n2) =>
        n1.CompareTo(n2) < 0;
    public static bool operator <=(Number n1, Number n2) =>
        n1.CompareTo(n2) <= 0;
    public static bool operator >=(Number n1, Number n2) =>
        n1.CompareTo(n2) <= 0;
}
```
Использование:
```csharp
var n1 = new Number { Val = 1 };
var n2 = new Number { Val = 10 };
bool bow = n1 > n2;
bool men = n2 <= n1;
```

## Ленивое создание объектов

Для отложенного создания объектов применяется обобщенный класс Lazy<>.

Полезно не только для уменьшения количества выделенной памяти под ненужные объекты, но и когда применяется затратный в плане ресурсов код, как вызов удаленного метода, обращение к базе данных.

Внутренний тип данных создается с симпользованием стандартного конструктора.

Пример ленивого создания объекта:
```csharp
class MyClass
{
    private Lazy<String> name = new Lazy<String>(() => "новая строка");
    public String GetName()
    {
        return name.Value; //значение с отложенной инициализацией
    }
}
public static void Main()
{
    MyClass my = new MyClass();
    Console.WriteLine(my.GetName()); //здесь размещение объекта в памяти
    Console.ReadLine();
}
```

## Сборка мусора

Чем больше объект существует в куче, тем выше вероятность того, что он таб будет оставаться. Каждый объект может находиться в: поколении 0 - новый размещенный в памяти объект, который еще никода не помечался как подлежащий сборке мусора; поколение 1 - объект, который пережил одну сборку мусора; поколение 2 - объект, которому удалось пережить более одной очистки. Поколения 0 и 1 - эфемерные.

До версии .NET 4.0 среда выполенения сборку мусора производила как "параммельную сборку мусора" - временная приостановка всех активных потоков внутри текущего процесса.

Но начиная с версии .NET 4.0 сборщик мусора производит как "фоновую сборку мусора" - фоновая сборка мусора для эфемерных поколений.

System.GC - класс взаимодействия со сборщикм мусора:
```csharp
Collect()           Немедленное выполение сборки мусора, можно указать номер поколения и режима сборки
GetGeneration()     Показывает поколение, к которому относится этот объект
GetTotalMemory()    Оценочный объем памяти в байтах, выделенный для кучи.
```
Пример применения:
```csharp
public static void Main()
{
    Console.WriteLine($"Размер кучи в байтах: {GC.GetTotalMemory(false)}");
    DataTable table = new DataTable();
    Console.WriteLine($"Поколение объекта: {GC.GetGeneration(table)}");
    GC.Collect(0,GCCollectionMode.Forced); //принудительно исследовать только поколение 0 и в немедленном режиме
    GC.WaitForPendingFinalizers(); //ждать окончания сборки
    Console.WriteLine($"Теперь поколение объекта: {GC.GetGeneration(table)}");
    Console.WriteLine($"Количества по поколениям: 0-{GC.CollectionCount(0)} 1-{GC.CollectionCount(1)} 2-{GC.CollectionCount(2)}");
    Console.ReadLine();
}
```
Специальный метод Finalize(), выполняемый при каждом уничтожении объекта сборщиком мусора:
```csharp
class MyClass
{
    ~MyClass() => Console.Beep();
}
```
Метод Dispose() для освобождения использовавшихся ресурсов:
```csharp
class MyClass : IDisposable
{
    public void Dispose()
    {
        Console.Beep();
    }
}
public static void Main()
{
    MyClass my = new MyClass();
    if (my is IDisposable)
        my.Dispose();
    Console.ReadLine();
}
```
Тоже самое но более грамотно (должен реализовывать интерфейс IDisposable, при выходе из области using вызывается Dispose):
```csharp
public static void Main()
{
    using (MyClass my = new MyClass())
    {
        Console.WriteLine(my.ToString());
    }
    Console.ReadLine();
}
```
Несколько объектов одного типа:
```csharp
public static void Main()
{
    using (MyClass my = new MyClass(), own = new MyClass())
    {
        Console.WriteLine(my.ToString());
    }
    Console.ReadLine();
}
```



