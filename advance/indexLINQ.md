# LINQ

LINQ - строго типизированный язык запросов, встроенный непосредственнов в язык самого C#. Используя его, можно создавать любое число выражений, которые выглядят и ведут себя подобно SQL запросам к базе данных. Но при этом этот запрос может применяться к любым хранилищам данных, и к тем, которые не имеют ничего общего с базами данных.

Язык C# использует следующие связанные с LINQ вещи:

- неявно типизированные локальные переменные.

- синтаксис инициализации объектов и коллекций.

- лямбда-выражения.

- расширяющие методы.

- анонимные типы.

Примняя LINQ, можно создавать прямо внутри языка программиования C# конструкции, которые называются выражениями запросо. Такие выражения запросов основаны на многочисленных операциях запросов, которые намеренно сделаны похожими внешне и по поведению на выражения SQL.

Выражения LINQ могут использоватся для взаимодействия с разнообразными типами данных - даже с теми, которы не имеют ничего общего с релиционными базами данных. LINQ это термин подхода доступа к данным. Разлиные обозначения запросов LINQ:

- LINQ to Objects. Применение запросов LINQ к массивам и коллекциям.

- LINQ to XML. Использование LINQ для манипулирования и запрашивания документов XML.

- LINQ to DataSet. Применение LINQ к объектам DataSet из ADO.NET.

- LINQ to Entities. Использование LINQ внутри API интерфейса ADO.NET Entity Framework.

- Parallel LINQ (PLINQ). Параллельная обработка данных, возвращаемых из запроса LINQ.

Выражение запроса LINQ строго типизировано. Значит, компилятор гарантирует корректность синтаксиса запросов.

## Состав LINQ

Основные сборки LINQ:
```csharp
System.Core.dll(System.Linq)       Определяет типы, представляющие основной API интерфейс LINQ. 
System.Data.DataSetExtensions.dll   Набор типов для интеграции типов ADO.NET - DataSet.
System.Xml.Linq.dll                 Функциональность для применения LINQ с данными документов XML - XML.
```
Для применения LINQ нужно написать директиву "using System.Linq".

## Массиивы и LINQ

Выражения LINQ значительно упрощают процесс работы с массивами.

Пример выражения для запроса элементов из массива с условиями и сортировкой:
```csharp
string[] names = {"Andrey", "Max", "Fedor 2", "1 Ivan", "Egor"};
IEnumerable<string> subset = from n in names
    where n.Contains(" ")
    orderby n
    select n;
foreach (var elem in subset)
{
    WriteLine($"Item: {elem}");
}
```
Существует синтаксис LINQ на расширяющих методах. Большинство операторов можно записать на любом из двух способов, но более сложные запросы будут требовать все-же использования выражений запросов.

Пример на расширяющих методах:
```csharp
string[] names = {"Andrey", "Max", "Fedor 2", "1 Ivan", "Egor"};
IEnumerable<string> subset = 
    names.Where(n => n.Contains(" "))
        .OrderBy(n => n)
        .Select(n => n);
foreach (var elem in subset)
{
    WriteLine($"Item: {elem}");
}
```
Тип результирующего набора немного отличается у обоих способов, хотя использовать их можно одинаково, так как производные от IEnumerable<T>. У первого - OrderedEnumerable<TElement, TKey>. У второго - System.Linq.WhereSelectEnumerableIterator.
    
Результирующие наборы LINQ могут быть представлены с применением порядочного количества типов из разнообразных пространств имен LINQ. Лещащий в основе тип не очевиден и даже напрямую не доступен в коде, тип может создаватся на этапе компиляции. Неявная типизация при работе с запросами LINQ значительно упрощает работу:

```csharp
string[] names = {"Andrey", "Max", "Fedor 2", "1 Ivan", "Egor"};
var subset = from n in names 
    where n.Contains(" ")
    orderby n
    select n;
foreach (var elem in subset)
{
    WriteLine($"Item: {elem}");
}
```

При захвате результатов запроса LINQ всегда необходимо использовать неявную типизацию, но действительное возвращаемое значение как правило имеет тип, реализующий интерфейс IEnumerable<T>. 

Класс System.Array получают в свое распоряжение на заднем плане множество обобщенных расширяющих методов, получаемых от служебного класса System.Linq.Enumerable.

## Отложенное или немедленное выполнение.

Выражения запросы LINQ имеют важную особенность - они в действительности не оцениваются до тех пор, пока не начнется итерация по последовательности в блоке foreach. Это называется отложенным выполнением. Преимущество такого подхода - возможность примененения одного и того же запроса LINQ многократно к тому же самому контейнеру и полной гарантией получения актуальных результатов.

Пример:
```csharp
string[] names = {"Andrey", "Max", "Fedor 2", "1 Ivan", "Egor"};
var subset = from n in names 
    select n;
//оператор LINQ здесь оценивается
foreach (var elem in subset)
{
    WriteLine($"Item: {elem}");
}
names[0] = "Pavel";
//оператор LINQ здесь снова оценивается
foreach (var elem in subset)
{
    WriteLine($"Item: {elem}");
}
```
Когда требуется оценить выражение LINQ за пределами логики foreach, можно вызывать любое количество расширяющих методов, определенных в типе Enumerable, такие как ToArray<T>, ToDictionary<TSource, TKey>, ToList<T>. Это приказывает выполнить запрос LINQ в момент их вызова для получения снимка данных, затем с этим снимком можно как угодно работать.

Пример немедленного выполенения:
```csharp
string[] names = {"Andrey", "Max", "Fedor 2", "1 Ivan", "Egor"};
string[] substr = (from n in names
    where n.Contains("M")
    select n).ToArray();
List<string> subList = (from n in names
    where n.Contains("A")
    select n).ToList();
```

## Возвращение результатов запроса LINQ

Можно в классе или структуре определить поле, значением которого будет результат запроса LINQ. Но для этого нужно использовать явную типизацию и результат запроса должны  быть статическим, не уровня экземпляра. И поэтому можно только таким образом получить результат, а это уродливо и не может быть нужно где либо:
```csharp
class MyClass
{
    private static string[] names = {"Andrey", "Max", "Fedor 2", "1 Ivan", "Egor"};
    private IEnumerable<string> sub = from n in names select n;
    public void PrintIts()
    {
        foreach (var item in sub)
        {
            WriteLine(item);
        }
    }
}
```
Это плохой пример.

Результат запроса из класса внешнему коду можно передать через подходящий тип IEnumerable<T> или IEnumerable. Пример:
```csharp
class MyClass
{
    private static string[] names = {"Andrey", "Max", "Fedor 2", "1 Ivan", "Egor"};
    public IEnumerable<string> GetNames()
    {
        IEnumerable<string> enumerable = from n in names
            where n.Contains(" ")
            select n;
        return enumerable;
    }
}
//использование:
MyClass m = new MyClass();
foreach (var item in m.GetNames()) //отложенная оценка выражения
    WriteLine(item);
```

Результат запроса из класса вшеншему коду можно передать используя строго пизированный массив в качестве возвращаемого значения. Пример:
```csharp
class MyClass
{
    private static string[] names = {"Andrey", "Max", "Fedor 2", "1 Ivan", "Egor"};
    public string[] GetNames()
    {
        var enumerable = from n in names
            where n.Contains(" ")
            select n;
        return enumerable.ToArray(); //немедленное выполнение
    }
}
//использование:
MyClass m = new MyClass();
foreach (var item in m.GetNames())
    WriteLine(item);
```

## Коллекции и LINQ

Тестовая коллекция - контейнер для примера:
```csharp
class Person
{
    public string Name { get; set; }
    public string Surname { get; set; }
    public int Age { get; set; }
    public static List<Person> GetTestList()
    {
        List<Person> persons = new List<Person>
        {
            new Person {Name = "And", Surname = "Fox", Age = 18},
            new Person {Name = "Zin", Surname = "Fox", Age = 16},
            new Person {Name = "Max", Surname = "Ivanov", Age = 25},
            new Person {Name = "Fax", Surname = "Kotov", Age = 32},
            new Person {Name = "Nik", Surname = "Vovov", Age = 22},
            new Person {Name = "Ivan I", Surname = "Romanov", Age = 99}
        };
        return persons;
    }
}
```

Применение выражения запроса LINQ по отношению к обобщенной коллекции не отличается от такового для простых массивов. Пример:
```csharp
List<Person> persons = Person.GetTestList();
var youngAPersons = from p in persons
    where p.Age < 20 && p.Surname == "Fox"
    select p;
foreach (var item in youngAPersons)
{
    WriteLine($"{item.Surname} {item.Name} {item.Age} лет");
}
```

Применение выражения запроса LINQ к необобщенным коллекциям возможна с использованием обобщенного расширяющего метода Enumerable.OfType<T>(). Пример:
```csharp
ArrayList myArrayList = new ArrayList()
{
    new Person {Name = "And", Surname = "Fox", Age = 18},
    new Person {Name = "Zin", Surname = "Fox", Age = 16},
    new Person {Name = "Max", Surname = "Ivanov", Age = 25}
};
var persons = myArrayList.OfType<Person>();
var youngAPersons = from p in persons
    where p.Age < 20 && p.Surname == "Fox"
    select p;
foreach (var item in youngAPersons)
{
    WriteLine($"{item.Surname} {item.Name} {item.Age} лет");
}
```

Этим методом еще можно фильтровать элементы в коллекции:

```csharp
ArrayList myArrayList = new ArrayList() { new Person(), 1, 4, "Test", 98};
var numbers = myArrayList.OfType<int>();
foreach (var elem in numbers)
{
    WriteLine(elem);
}
```

## Операции запроса LINQ

Распространенные операции запросов LINQ
```csharp
from, in - определение основы любого выражения LINQ, извлечение данных из контейнера
where - определение ограничений отностительно того, какие элементы извлечь из контейнера
select - выборка последовательности из контейнера
join, on, equals, into - соедниения на основе указанного ключа
orderby, ascending, descending - упорядочить результирующий набор по возрастанию или убыванию
group, by - множество данных сгруппированных по значению
```

Статический класс System.Linq.Enumerable предлагает набор методов, доступных как расширяющие методы. Они могу трансформировать результирующий набор данных, извлечения одиночных элементов или разообразные операции над множествами плюс агрегация результатов.

Тестовый класс с массивом для примера:
```csharp
class Info
{
    public string Name { get; set; }
    public int Number { get; set; }
    public override string ToString() => $"Name = {Name} Num = {Number}";
    public static Info[] GetTestArr()
    {
        return new[]
        {
            new Info {Name = "Coffee", Desc = "Coggds gggs ss", Number = 1},
            new Info {Name = "Milk", Desc = "Mily Way dss", Number = 6},
            new Info {Name = "Tea", Desc = "This Tea bag is", Number = 9},
            new Info {Name = "White Birds", Desc = "Lorem is not live", Number = 7},
            new Info {Name = "Oats", Desc = "Test Oats ds", Number = 99}
        };
    }
}
```
Базовый синтаксис выборки:
```csharp
var результат = from элемент in контейнер select элемент;
```
Пример:
```csharp
Info[] infos = Info.GetTestArr();
var all = from p in infos select p.Name; //только имена товаров
foreach (var el in all)
{
    WriteLine(el);
}
```
Получение подмножества данных:
```csharp
var результат = from элемент in контейнер where выражение select элемент;
```
Пример:
```csharp
Info[] infos = Info.GetTestArr();
var all = from p in infos where p.Number>3 select p.Name;
foreach (var el in all)
{
    WriteLine(el);
}
```
Проецирование новых типов данных из существующих источников данных:
```csharp
Info[] infos = Info.GetTestArr();
var all = from p in infos select new {p.Name, p.Desc};
foreach (var el in all)
{
    WriteLine(el.Name + " " + el.Desc);
}
```
Метод, возвращающий результат выражения LINQ в виде массива:
```csharp
static Array GetArray(Info[] infos)
{
    var all = from p in infos select new {p.Name, p.Desc};
    return all.ToArray();
}
//использование
Info[] infos = Info.GetTestArr();
var all = GetArray(infos);
foreach (var el in all)
{
    WriteLine(el);
}
```
Подсчет количества с использование класса Enumerable:
```csharp
Info[] infos = Info.GetTestArr();
int count = (from p in infos select p).Count();
```
Изменение порядка следования элементов на противоположный:
```csharp
Info[] infos = Info.GetTestArr();
var all = (from p in infos select p).Reverse();
```
Выражения сортировки:
```csharp
Info[] infos = Info.GetTestArr();
var all = from p in infos orderby p select p; //по возрастанию
var all2 = from p in infos orderby p descending select p; //по убыванию
```


