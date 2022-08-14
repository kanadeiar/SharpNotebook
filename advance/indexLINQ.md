# LINQ

LINQ - строго типизированный язык запросов, встроенный непосредственнов в язык самого C#. Используя его, можно создавать любое число выражений, которые выглядят и ведут себя подобно SQL запросам к базе данных. Но при этом этот запрос может применяться к любым хранилищам данных, и к тем, которые не имеют ничего общего с базами данных.

Язык C# использует следующие связанные с LINQ вещи:

- неявно типизированные локальные переменные.

- синтаксис инициализации объектов и коллекций.

- лямбда-выражения.

- расширяющие методы.

- анонимные типы.

Особенности выражений запросов LINQ и их внутреннего представления:

- выражения запросов создаются с применением разнообразных операций запросов C#.

- операции запросов - это сокращенное обозрачение для вызова расширяющих методов, определенный в типе System.Linq.Enumerable.

- многие методы класса требудт передачи делегатов в качестве параметров.

- любой метод, ожидающий параметр тип делегата, может принимать лямбда-выржаение.

- лямбда-выражения являются лишь замаскированными анонимными методами.

- анонимные методы представляют собой соращенные обозначения для размещения экземпляра никоуровневого делегата и ручного построения целевого метода делегата.

Применяя LINQ, можно создавать прямо внутри языка программиования C# конструкции, которые называются выражениями запросо. Такие выражения запросов основаны на многочисленных операциях запросов, которые намеренно сделаны похожими внешне и по поведению на выражения SQL.

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
    WriteLine($"Item: {elem}");
```

Тип результирующего набора немного отличается у обоих способов, хотя использовать их можно одинаково, так как производные от IEnumerable<T>. У первого - OrderedEnumerable<TElement, TKey>. У второго - System.Linq.WhereSelectEnumerableIterator.
    
Результирующие наборы LINQ могут быть представлены с применением порядочного количества типов из разнообразных пространств имен LINQ. Лещащий в основе тип не очевиден и даже напрямую не доступен в коде, тип может создаватся на этапе компиляции. 

Неявная типизация при работе с запросами LINQ значительно упрощает работу:

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

В расширяющих методах можно опускать последний оператор. 

Пример:
```csharp
string[] names = {"Andrey", "Max", "Fedor 2", "1 Ivan", "Egor"};
IEnumerable<string> subset = names.Where(n => n.Contains(" "));
```
Можно использовать сразу создание новых типов или анонимных типов данных. 

Примеры:
```csharp
var report = db.Where(e => e.Age >= 20 && e.Salary > 5)
    .Select(e => new NewWorker{ q.Name, q.Salary });
var report2 = db.Where(w => w.Age > 20)
    .OrderBy(w => w.Name)
    .ThenByDescending(w => w.Age) //дополнительная сортировка по убыванию
    .Select(e => new { e.Name, e.Salary });
```

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

Результат запроса из класса вшеншему коду можно передать используя строго тизированный массив в качестве возвращаемого значения. Пример:
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
var all = from p in infos select new {p.Name, p.Desc}; //проецирование нового типа данных
foreach (var el in all)
{
    WriteLine(el.Name + " " + el.Desc);
}
```
Метод, возвращающий результат выражения LINQ в виде массива:
```csharp
static Array GetArray(Info[] infos)
{
    var all = from p in infos select new {p.Name, p.Desc}; //проецирование нового типа данных
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
Класс Enumerable поддерживает несколько расширяющих методов для работы с двумя и более коллекциями одновременно с целью нахождения их объединений, разностей, конкатенаций и пересечений данных.


- пример разности двух контейнеров метод Except():
```csharp
List<string> name1 = new List<string> {"And", "Vlad", "Sergey"};
List<string> name2 = new List<string> {"Vlad", "Sergey", "Max"};
var diff = (from n in name1 select n)
    .Except(from n in name2 select n);
foreach (var el in diff)
{
    WriteLine(el); //And
}
```
- пример общие элементы данных в наборе контейнеров метод Intersect():
```csharp
List<string> name1 = new List<string> {"And", "Vlad", "Sergey"};
List<string> name2 = new List<string> {"Vlad", "Sergey", "Max"};
var intersect = (from n in name1 select n)
    .Intersect(from n in name2 select n);
foreach (var el in intersect)
{
    WriteLine(el); //Vlad Sergey
}
```
- пример включеие всех членов множества запросов LINQ метод Union():
```csharp
List<string> name1 = new List<string> {"And", "Vlad", "Sergey"};
List<string> name2 = new List<string> {"Vlad", "Sergey", "Max"};
var union = (from n in name1 select n)
    .Union(from n in name2 select n);
foreach (var el in union)
{
    WriteLine(el); //And Vlad Sergey Max
}
```
- пример прямой конкатенацией результирующих наборов LINQ метод Concat():
```csharp
List<string> name1 = new List<string> {"And", "Vlad", "Sergey"};
List<string> name2 = new List<string> {"Vlad", "Sergey", "Max"};
var concat = (from n in name1 select n)
    .Concat(from n in name2 select n);
foreach (var el in concat)
{
    WriteLine(el); //And Vlad Sergey Vlad Sergey Max
}
```
Пример метода удаления дубликатов - расширяющий метод Distinct():
```csharp
List<string> name1 = new List<string> {"And", "Vlad", "Sergey"};
List<string> name2 = new List<string> {"Vlad", "Sergey", "Max"};
var concat = ((from n in name1 select n)
    .Concat(from n in name2 select n)).Distinct();
foreach (var el in concat)
{
    WriteLine(el); //And Vlad Sergey Max
}
```
Операции агрегирования могут использоватся для выполнения надо результирующим набором разообразных операций агрегирования.

- Count - количество.

- Average - среднее значение.

- Max - самое большое значение.

- Min - самое малое значение.

- Sum - сумма значений.

- Take - первые значения.

- Skip - пропуск значений.

Примеры:
```csharp
int[] arrInts = {2, -3, 4, 9, 0, 10};
WriteLine($"Max = {(from i in arrInts select i).Max()}");
WriteLine($"Min = {(from i in arrInts select i).Min()}");
WriteLine($"Average = {(from i in arrInts select i).Average()}");
WriteLine($"Sum = {(from i in arrInts select i).Sum()}");
var arr1=(from i in arrInts select i).Take(2);
foreach (var el in arr1)
    WriteLine($"{el}");
var arr2=(from i in arrInts select i).Skip(2);
foreach (var el in arr2)
    WriteLine($"{el}");
```

## Группировка

Примеры:
```csharp
var groups = from user in users
             group user by user.LastName;
foreach (var g in groups)
{
    Console.Write($"{g.Key}: ");
    foreach (var t in g)
        Console.Write($"{t.FirstName} ");
    Console.WriteLine();
}
```
Еще пример:
```csharp
var group1 = users.GroupBy(p=>p.LastName)
    .Select(g=>new
    {
        Family = g.Key,
        Count = g.Count(),
        Names = g.Select(p=>p),
    });
foreach (var el in group1)
{
    Console.Write($"{el.Family} {el.Count} шт.: ");
    foreach (var t in el.Names)
        Console.Write($"{t.FirstName} ");
    Console.WriteLine();
}
```

Проход по коллекциям - детям:
```csharp
class Child
{
    public string Name { get; set; }
    public int Age { get; set; }
}
class Person : Child
{
    public List<Child> Children { get; set; }
}
static void Main(string[] args)
{
    var persons = new List<Person>();
    //дети есть с десятилетним возрастом
    var with10AgeChild = persons.Where(p => p.Children.Select(c => c.Age).Contains(10));
    //есть маленькие дети
    var withManyKindChild = persons.Where(p => p.Children.Count(c => c.Age < 8) > 0);
    //есть хотяб один малельний ребенок
    var withOneKind = persons.Where(p => p.Children.FirstOrDefault(c => c.Age < 8) != null);
```


## Внутренности операторов запросов LINQ.

В действительности комплиятор C# транслирует все операции запросов LINQ в вызовы методов класса Enumerable. Огромное количество методов класса Enumerable протипированы для приема делегатов в качестве аргументов, требуют обобщенный делегат Func<>. Использование операций запросов LINQ является самым простым способом построения запросов LINQ. 

Построение операций запросов LINQ является простым способом построения выражений LINQ, но можно применять еще другие подходы, более сложные. Преимущество использования операций запросов C# при построении выражений запросов заключается в том, что делегаты Func<> и вызовы методов Enumerable остаются вне поля зрения и внимание, так как выполнение трансляции возлагается на компилятор C#. Это наиболее распространенный и простой подход.

Пример выражения запроса с применением операций запросов:
```csharp
string[] names = { "Andrey", "Max", "Fedor 2", "1 Ivan", "Egor" };
var substr = from n in names
             select n;
```
Можно строить более сложные запросы с использованием типа Enumerable и лямбда-выражений. Это прямое использование расширяющих методов Enumerable.
```csharp
string[] names = { "Andrey", "Max", "Fedor 2", "1 Ivan", "Egor" };
var substr = names.Where(n => n.Contains("a"))
    .OrderBy(n => n)
    .Select(n => n);
```
Запрос можно сделать с выделением каждого шага в отдельный фрагмент:
```csharp
string[] names = { "Andrey", "Max", "Fedor 2", "1 Ivan", "Egor" };
var namesA = names.Where(n => n.Contains("a"));
var ordered = namesA.OrderBy(n => n);
var substr = ordered.Select(n => n);
```
Можно строить выражения запросов LINQ с применением анонимных методов. Пример вручную созданные делегаты Func<>, используемые методами Where(), OrderBy() и Select() класса Enumerable:
```csharp
string[] names = { "Andrey", "Max", "Fedor 2", "1 Ivan", "Egor" };
Func<string, bool> filter = delegate (string name) { return name.Contains(" "); };
Func<string, string> itemSel = delegate (string s) { return s; };
var substr = names.Where(filter)
    .OrderBy(itemSel)
    .Select(itemSel);
foreach (var item in substr)
{
    WriteLine(item);
}
```

Можно совсем отказаться от использования синтаксиса лямбда-выражений и анонимных методов и напрямую создавать цели делегатов для каждого типа Func<>, создавать низкоуровневые делегаты. Пример такого способа:

```csharp
    string[] names = { "Andrey", "Max", "Fedor 2", "1 Ivan", "Egor" };
    Func<string, bool> filter = new Func<string, bool>(Filter);
    Func<string, string> itemSel = new Func<string, string>(ProcIt);
    var substr = names.Where(filter)
        .OrderBy(itemSel)
        .Select(itemSel);
//Цели делегатов
public static bool Filter(string name) { return name.Contains(" "); }
public static string ProcIt(string name) { return name; }
```

## Чтения файла и LINQ

Пример (функции):
```csharp
var sum = File.ReadAllText("data.txt")
          .Split(' ')
          .Select(ConvetrStringToInt)
          .Where(Search)
          .ToArray()
          .Sum();
...
static bool Search(int Item)
{
    return Item % 2 == 0;
}

static int ConvetrStringToInt(string Str)
{
    return Convert.ToInt32(Str);
}
```
Еще пример (делегаты):
```csharp
var sum = File.ReadAllText("data.txt")
          .Split(' ')
          .Select(delegate (string Str)
          {
              return Convert.ToInt32(Str);
          })
          .Where(delegate (int Item)
          {
              return Item % 2 == 0;
          })
          .ToArray()
          .Sum();
```
Еще пример (лямбды):
```csharp
var sum = System.IO
              .File.ReadAllText("data.txt")
              .Split(' ')
              .Select(int.Parse)
              .Where(e => e % 2 == 0)
              .ToArray()
              .Sum();
```

