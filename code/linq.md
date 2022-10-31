# LINQ

Набор технологий LINQ (Language Integrated Query — язык интегрированных запросов), предоставляет краткий, симметричный и строго типизированный способ доступа к широкому разнообразию хранилищ данных.

LINQ - строго типизированный язык запросов, встроенный непосредственнов в язык самого C#. Используя его, можно создавать любое число выражений, которые выглядят и ведут себя подобно SQL запросам к базе данных. Но при этом этот запрос может применяться к любым хранилищам данных, и к тем, которые не имеют ничего общего с базами данных.

Применяя LINQ, можно создавать прямо внутри языка программиования C# конструкции, которые называются выражениями запросов. Такие выражения запросов основаны на многочисленных операциях запросов, которые намеренно сделаны похожими внешне и по поведению на выражения SQL.

Выражения LINQ могут использоватся для взаимодействия с разнообразными типами данных - даже с теми, которы не имеют ничего общего с релиционными базами данных.

Выражение запроса LINQ строго типизировано. Значит, компилятор гарантирует корректность синтаксиса запросов.

## Массиивы и LINQ

Выражения LINQ значительно упрощают процесс работы с массивами.

Пример выражения LINQ для запроса элементов из массива с условиями и сортировкой:

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

Существует синтаксис LINQ на расширяющих методах. 

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

Неявная типизация при работе с запросами LINQ значительно упрощает работу:

```csharp
string[] names = { "Andrey", "Max", "Fedor 2", "1 Ivan", "Egor" };
var subset = names
    .Where(x => x.Contains(" "))
    .OrderBy(x => x)
    .Select(x => x);
foreach (var elem in subset)
{
    Console.WriteLine($"Item: {elem}");
}
```

При захвате результатов запроса LINQ всегда необходимо использовать неявную типизацию, но действительное возвращаемое значение как правило имеет тип, реализующий интерфейс IEnumerable<T>. 

## Отложенное выполнение.

Выражения запросы LINQ имеют важную особенность - они в действительности не оцениваются до тех пор, пока не начнется итерация по последовательности в блоке foreach. Это называется отложенным выполнением. Преимущество такого подхода - возможность примененения одного и того же запроса LINQ многократно к тому же самому контейнеру и полной гарантией получения актуальных результатов.

Пример:

```csharp
string[] names = { "Andrey", "Max", "Fedor 2", "1 Ivan", "Egor" };
var subset = names
    .Select(x => x);
//оператор LINQ здесь оценивается
foreach (var elem in subset)
{
    Console.WriteLine($"Item: {elem}");
}
names[0] = "Pavel";
//оператор LINQ здесь снова оценивается
foreach (var elem in subset)
{
    Console.WriteLine($"Item: {elem}");
}
```

Когда требуется оценить выражение LINQ за пределами логики foreach, можно вызывать любое количество расширяющих методов, определенных в типе Enumerable, такие как ToArray<T>, ToDictionary<TSource, TKey>, ToList<T>. Это приказывает выполнить запрос LINQ в момент их вызова для получения снимка данных, затем с этим снимком можно как угодно работать.

Пример немедленного выполенения:

```csharp
string[] names = { "Andrey", "Max", "Fedor 2", "1 Ivan", "Egor" };
string[] substr = (from n in names
                    where n.Contains("M")
                    select n).ToArray();
List<string> subList = names
    .Where(x => x.Contains("A"))
    .ToList();
```

## Возвращение результатов запроса LINQ

Можно в классе или структуре определить поле, значением которого будет результат запроса LINQ. Результат запроса из класса внешнему коду можно передать через подходящий тип IEnumerable<T> или IEnumerable. 

Пример:

```csharp
class MyClass
{
    private static string[] names = { "Andrey", "Max", "Fedor 2", "1 Ivan", "Egor" };
    public IEnumerable<string> GetNames()
    {
        IEnumerable<string> enumerable = names
            .Where(x => x.Contains(" "))
            .Select(x => x);
        return enumerable;
    }
}
//использование:
MyClass m = new MyClass();
foreach (var item in m.GetNames()) //отложенная оценка выражения
    WriteLine(item);
```

## Фильтрация LINQ по типу

Можно использовать методы OfType<T> для фильтрации элементов коллекции, которые удовлетворяют заданному в параметре-типе типу.

```csharp
object[] values = { "Andrey", 1, 2, 1.4, 9L };
var ints = values.OfType<int>();
```

## Операции LINQ

Базовый синтаксис выборки:

```csharp
var результат = from элемент in контейнер select элемент;
```

Структура для примеров:

```csharp
struct Info
{
    public string Name { get; set; } = string.Empty;
    public int Number { get; set; } = default;
    public int Desc { get; set; } = default;
    public Info() { }
}
```

Пример:

```csharp
var all = from p in infos select p.Name; //только имена товаров
var infos = Enumerable.Range(1, 10)
    .Select(x => new Info { Name = $"N{x}", Number = x, Desc = x });
var all = infos
    .Select(x => x.Name); //только имена товаров
foreach (var el in all)
    Console.WriteLine(el);
```

Получение подмножества данных:

```csharp
var результат = from элемент in контейнер where выражение select элемент;
```

Пример:

```csharp
var all = from p in infos where p.Number>3 select p.Name;
var infos = Enumerable.Range(1, 10)
    .Select(x => new Info { Name = $"N{x}", Number = x, Desc = x });
var all = infos
    .Where(x => x.Number > 3)
    .Select(x => x.Name); //только имена товаров
foreach (var el in all)
    Console.WriteLine(el);
```

Проецирование новых типов данных из существующих источников данных:

```csharp
var all = from p in infos select new {p.Name, p.Desc}; //проецирование нового типа данных
var infos = Enumerable.Range(1, 10)
    .Select(x => new Info { Name = $"N{x}", Number = x, Desc = x });
var all = infos
    .Select(x => new { x.Name, x.Desc }); //проецирование нового типа данных
foreach (var el in all)
    Console.WriteLine($"{el.Name} {el.Desc}");
```

Подсчет количества с использование класса Enumerable:

```csharp
int count = (from p in infos select p).Count();
var infos = Enumerable.Range(1, 10)
    .Select(x => new Info { Name = $"N{x}", Number = x, Desc = x });
var count = infos.Count();
```

В C# есть возможность получить количество элементов в коллекции без актуального перечисления элементов списка. Для этого следует использовать расширяющий метод LINQ TryGetNonEnumeratedCount().

```csharp
var infos = Enumerable.Range(1, 10)
    .Select(x => new Info { Name = $"N{x}", Number = x, Desc = x });
var _ = infos.TryGetNonEnumeratedCount(out int count);
Console.WriteLine(count);
```

Изменение порядка следования элементов на противоположный:

```csharp
var all = infos.Reverse();
```

Выражения сортировки:

```csharp
var infos = Enumerable.Range(1, 10)
    .Select(x => new Info { Name = $"N{x}", Number = x, Desc = x });
var all1 = infos.OrderBy(x => x.Name);
var all2 = infos.OrderByDescending(x => x.Desc);
```

## Манипуляции коллекциями LINQ

Класс Enumerable поддерживает несколько расширяющих методов для работы с двумя и более коллекциями одновременно с целью нахождения их объединений, разностей, конкатенаций и пересечений данных.

- пример разности двух контейнеров метод Except():

```csharp
List<string> name1 = new() { "And", "Vlad", "Sergey" };
var name2 = new List<string> { "Vlad", "Sergey", "Max" };
var diff = name1
    .Except(name2);
foreach (var el in diff)
{
    Console.WriteLine(el); //And
}
```

- пример общие элементы данных в наборе контейнеров метод Intersect():

```csharp
List<string> name1 = new() {"And", "Vlad", "Sergey"};
var name2 = new List<string> {"Vlad", "Sergey", "Max"};
var intersect = (from n in name1 select n)
    .Intersect(from n in name2 select n);
foreach (var el in intersect)
{
    WriteLine(el); //Vlad Sergey
}
```

- пример включеие всех членов множества запросов LINQ метод Union():

```csharp
List<string> name1 = new() { "And", "Vlad", "Sergey" };
var name2 = new List<string> { "Vlad", "Sergey", "Max" };
var union = name1.Select(x => x)
    .Union(name2.Select(x => x));
foreach (var el in union)
{
    Console.WriteLine(el); //And Vlad Sergey Max
}
```

- пример прямой конкатенацией результирующих наборов LINQ метод Concat():

```csharp
var name1 = new List<string> {"And", "Vlad", "Sergey"};
var name2 = new List<string> {"Vlad", "Sergey", "Max"};
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

## Агрегирование

Операции агрегирования могут использоватся для выполнения надо результирующим набором разообразных операций агрегирования.

```csharp
var arrInts = new[] { 2, -3, 4, 9, 0, 10 };
Console.WriteLine($"Max = {(from i in arrInts select i).Max()}");
Console.WriteLine($"Min = {arrInts.Min()}");
Console.WriteLine($"Average = {(from i in arrInts select i).Average()}");
Console.WriteLine($"Sum = {arrInts.Select(x => x).Sum()}");
var arr1 = (from i in arrInts select i).Take(2);
foreach (var el in arr1)
    Console.Write($"{el} ");
Console.WriteLine();
var arr2 = arrInts.Skip(2);
foreach (var el in arr2)
    Console.Write($"{el} ");
```

## Группировка

Разбивает последовательность на подпоследовательности.

Пример простой группировки данных:

```csharp
record class Info(string Name, int Number, int Desc);
var infos = Enumerable.Range(1, 10)
    .Select(x => new Info($"N{x}", x, x));
var group1 = infos.GroupBy(p => p.Name);
foreach (var el in group1)
{
    Console.WriteLine($"{el.Key} - {el.Count()} шт.:");
    foreach (var x in el)
    {
        Console.WriteLine($"{x.Name} {x.Number}");
    }
    Console.WriteLine();
}
```

Пример группировки данных с проецированием:

```csharp
record class Info(string Name, int Number, int Desc);
var infos = Enumerable.Range(1, 10)
    .Select(x => new Info($"N{x}", x, x));
var group1 = infos.GroupBy(p => p.Name)
    .Select(g => new
    {
        Name = g.Key,
        Count = g.Count(),
        Elems = g.Select(p => p),
    });
foreach (var el in group1)
{
    Console.Write($"{el.Name} {el.Count} шт.: ");
    foreach (var t in el.Elems)
        Console.Write($"{t.Name} ");
    Console.WriteLine();
}
```

Возможно делать вложенные запросы.

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
var persons = Enumerable.Range(1, 5)
    .Select(x => new Person { Name = "Родитель", Age = x + 20, Children = new List<Child> { new Child { Name = "Ребенок", Age = x + 20 } } });
//проецирование нового типа
var newTypes = persons.GroupBy(x => x.Name).Select(x => new
{
    Name = x.Key,
    Count = x.Count(),
    Childs = x.Select(a => a),
});
//дети есть с десятилетним возрастом
var with10AgeChild = persons.Where(p => p.Children.Select(c => c.Age).Contains(10));
//есть маленькие дети
var withManyKindChild = persons.Where(p => p.Children.Count(c => c.Age < 8) > 0);
//есть хотяб один малельний ребенок
var withOneKind = persons.Where(p => p.Children.FirstOrDefault(c => c.Age < 8) != null);
```

## Сведение объектов

С помощью метода SelectMany() можно производить сведение коллекций в одну или проецировать все элементы коллекций в другой тип.

Пример простой:

```csharp
record class Info(string Name, int Number, int Desc);
var infos = Enumerable.Range(1, 10)
    .Select(x => new { Name = x, Infos = new List<Info> { new Info($"N{x}", x, x) } });
var allinfos = infos.SelectMany(x => x.Infos);
foreach (var el in allinfos)
{
    Console.Write($"{el.Name} {el.Number} ");
}
```

Пример с проецированием:

```csharp
var infos = Enumerable.Range(1, 10)
    .Select(x => new { Name = $"Имя{x}", Infos = new List<Info> { new Info($"N{x}", x, x) } });
var converted = infos.SelectMany(x => x.Infos, (x, y) => new { Name = x.Name, InfoName = y.Name });
foreach (var el in converted)
{
    Console.Write($"{el.Name} {el.InfoName} ");
}
```

## Пагинация данных

Есть возможность в LINQ осуществлять пагинацию даных без использоания методов Take() и Skip() вместе. Можно использовать в методе Take() типы Range. Пример:

```csharp
var names = new[] { "Иван", "Петр", "Сидор", "Вася", "Павел", "Дима" };
var three = names.Take(..3); //первые три
var skipping = names.Take(3..); //пропуск первых трех
var last = names.Take(^2..); //поледние два
var skip = names.Take(..^2); //пропуск последних двух
var skipTake = names.Take(2..4); //пропуск двух, забор двух
```

В языке C# в LINQ существует возможность разбивать массивы по страницам. Для этого следует использовать метод Chunk(). 

```csharp
var names = new[] { "Иван", "Петр", "Сидор", "Вася", "Павел", "Дима" };
var chunks = names.Chunk(size: 3);
foreach (var chunk in chunks)
{
    Console.Write($"Страница: ");
    foreach (var e in chunk)
    {
        Console.Write($"{e} ");
    }
    Console.WriteLine();
}
```

## Значение по умолчанию

Есть возможность указать значение по умолчанию, возвращаемое методами FirstOrDefault(), LastOrDefault(), SingleOrDefault(). Для этого значение нужно указать в параметрах метода.

Есть возможность указать значение по умолчанию, возвращаемое, если коллекция пуста - метод DefaultIfEmpty(). В параметре метода нужно указать значение по умолчанию.

Пример:

```csharp
var names = new[] { "Иван", "Петр", "Сидор", "Вася", "Павел", "Дима" };
var result = names.DefaultIfEmpty("нет").FirstOrDefault(x => x.StartsWith("test"));
```

## LINQ при чтении файлов

Пример:

```csharp
var sum = System.IO
    .File.ReadAllText("data.txt")
    .Split(' ')
    .Select(int.Parse)
    .Where(e => e % 2 == 0)
    .ToArray()
    .Sum();
```

