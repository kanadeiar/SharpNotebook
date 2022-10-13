# Чтение EF

## Получение записей данных

Можно применять метод ToQueryString() для получения текста SQL запроса, который будет исполнятся платформой EF Core.

Для получения всех записей в простой манере нужно использовать свойство DbSet<T> напрямую без использования LINQ конструкций и далее для немедленного получения результатов добавить метод ToList(). 

Пример:

```csharp
// простейшее
var samples = context.Samples;
foreach (var e in samples)
{
    Console.WriteLine($"{e.Name} {e.AdvancedName}");
}
// очистка контекста
context.ChangeTracker.Clear();
var arr = context.Samples.ToArray();
foreach (var e in arr) // clean
{
    Console.WriteLine($"{e.Name} {e.AdvancedName}");
}
```

### Фильтрация

Для отфильтровывания получаемых из базы данных записей следует применять метод Where(). Множественное примененение методов Where() друг за другом допускается.

Пример:

```csharp
// простейшая фильтрация
var query = context.Samples.Where(x => x.IsTest == true).AsQueryable();
foreach (var e in query)
{
    Console.WriteLine($"{e.Name} {e.AdvancedName}");
}
context.ChangeTracker.Clear();
// обычная
query = context.Samples.Where(x => x.Name.StartsWith("Test")).AsQueryable();
query = query.Where(x => x.IsTest == true);
foreach (var e in query)
{
    Console.WriteLine($"{e.Name}");
}
context.ChangeTracker.Clear();
// ускоренная
var samples = context.Samples.Where(x => !string.IsNullOrWhiteSpace(x.Name));
foreach (var e in samples)
{
    Console.WriteLine($"{e.Name}");
}
```

### Сортировка

Для сортировки получаемых из базы данных значений следует применять методы OrderBy(), OrderByDescending(). Если нужна дополнительная сортировка подэлементов, то применять нужно метод ThenBy() и ThenByDescending().

Пример:

```csharp
var samples = context.Samples.OrderBy(x => x.Name).ThenByDescending(x => x.AdvancedName);
foreach (var e in samples)
{
    Console.WriteLine($"{e.Name} {e.AdvancedName}");
}
```

Для сортировки в обратном порядке всех элементов можно применять метод Reverse().

Пример:

```csharp
var samples = context.Samples.OrderBy(x => x.Name).ThenBy(x => x.AdvancedName).Reverse();
foreach (var e in samples)
{
    Console.WriteLine($"{e.Name} {e.AdvancedName}");
}
```

### Пагинация

Можно разбивать получаемые данные постранично - производить пагинацию данных с помощью методов Skip() и Take().

Пример:

```csharp
var samples = context.Samples.Skip(2).Take(3).ToList();
foreach (var e in samples)
{
    Console.WriteLine($"{e.Name} {e.AdvancedName}");
}
```

### Получение отдельных элементов

Для получения отдельной записи из базы данных существует множество методов, а именно:

First() - получение первой записи, что соответствует запросу. Если запись не найдена, то выбрасывается исключение.

FirstOrDefault() - получение первой записи соответствующей запросу, при этом не выбрасывается исключение если записи нет, но вертается значение null.

Single() - получение одной записи из базы данных, соответствующей запросу. Если запись не найдена или найдено более одной записи, то выбрасывается исключение.

SingleOrDefault() - получение одной записи соответствующей запросу, при этом не выбрасывается исключение если записи нет или найдено более одной записи, но вертается значение null.

Last() - получение последней записи, что соответствует запросу. Если запись не найдена, то выбрасывается исключение.

LastOrDefault() - получение последней записи соответствующей запросу, при этом не выбрасывается исключение если записи нет, но вертается значение null.

Find() - получение одной записи, значение первичного ключа которой равен значению, переданному в параметрах этого метода

Пример:

```csharp
var one = context.Samples.Where(x => x.Id == 1).First();
var two = context.Samples.FirstOrDefault(x => x.Id == 1);
var third = context.Samples.Find(12);
context.ChangeTracker.Clear();
```

### Агрегация данных

EF Core поддерживает методы агрегации данных, выполняемых на серверной стороне. Это такие методы как Max(), Min(), Count(), Average(). 

Пример:

```csharp
var count = context.Samples.Count(x => x.IsTest);
var one = context.Samples.Max(x => x.Id);
var two = context.Samples.Average(x => x.Id);
```

### Получение результата от записей по критерию

С помощью методов Any() и All() можно получить информацию о результате нахождения хотябы одной записи по критерию (Any) или все записи соответствуют критерию (All).

Пример:

```csharp
var yes = context.Samples.Any(x => x.IsTest);
var no = context.Samples.All(x => x.Id == 1);
```

### Получение результата из хранимых процедур

Пример созданной хранимой процедуры в базе данных:

```sql
CREATE PROCEDURE [dbo].[GetName]
@Id int,
@name nvarchar(50) output
AS
SELECT @name = Name from dbo.MySample where Id = @Id
```

Пример работы с этой хранимой процедурой:

```csharp
var parameterId = new SqlParameter
{
    ParameterName = "@Id",
    SqlDbType = SqlDbType.Int,
    Value = 1,
};
var parameterName = new SqlParameter
{
    ParameterName = "@Name",
    SqlDbType = SqlDbType.NVarChar,
    Size = 50,
    Direction = ParameterDirection.Output,
};
var result = context.Database.ExecuteSqlRaw("EXEC [dbo].[GetName] @Id, @Name OUTPUT", parameterId, parameterName);
```

### Связанные данные

Навигационные свойства в сущности позволяют загружать связанные данные этой сущности. Данные могут быть загружены разными способами. При загрузке этих связанных сущностей ChangeTracker автоматически будет их отслеживать. 

#### Энергичная загрузка

Энергичная загрузка подразумевает загрузку связанных данных из нескольких таблиц базы данных за один вызов. EF Core все делает автоматически. Используются методы Include() и ThenInclude() для путешествия по навигационным свойствам сущности в LINQ запросах. 

Если отношения сущностей обязательны, то LINQ создает запрос внутреннего объединения таблиц. Если же отношения сущностей необязательны - запрос левого простого объединения таблиц.

Пример:

```csharp
var sampleWithMakes = context.Samples.Include(x => x.MakeNavigation).ToList();
context.ChangeTracker.Clear();
var sampleWithDrivers = context.Samples.Include(x => x.Drivers).ThenInclude(d => d.SampleDrivers).ToList();
context.ChangeTracker.Clear();
```

Внутрь метода Include() можно передавать методы сортировки, выборки и постраничной разбивки для того, чтобы отсортировать, отфильтровать или постранично разбить получаемые связанные данные.

Пример:

```csharp
var sampleWithMakes = context.Makes
    .Include(x => x.Samples.Where(x => x.IsTest).OrderBy(x => x.Name).Skip(1).Take(2))
    .ToList();
context.ChangeTracker.Clear();
```

Можно заставить EF Core разделить большой запрос на несколько подзапросов, применив метод AsSplitQuery(), и потом уже EF Core подулючет связанные данные. Это может повысить производительность при больших запросах.

Пример:

```csharp
var sampleWithMakes = context.Makes.AsSplitQuery()
    .Include(x => x.Samples.Where(x => x.IsTest).OrderBy(x => x.Name).Skip(1).Take(2))
    .ToList();
context.ChangeTracker.Clear();
```

Для получения данных из таблиц сущностей связанных по типу "многие-ко-многим" следует применять запросы, проходящие через связанную таблицу.

Пример:

```csharp
var sampleAndDrivers = context.Samples.Include(x => x.Drivers).Where(x => x.Drivers.Any());
context.ChangeTracker.Clear();
```

#### Явная загрузка

Такая загрузка связанных данных по навигационному свойству производится после того, как объект уже загружен. 

Начинать запрос следует с метода Entry() у производного от DbContext объекта. При запросе справочного навигационного свойства нужно использовать метод Reference(). При запросе свойства навигации коллекции нужно использовать метод Collection(). Запрос откладывается до тех пор, пока не будет вызваны методы Load(), ToList() или агрегатная функция.

Пример:

```csharp
var sample = context.Samples.First(x => x.Id == 1);
context.Entry(sample).Reference(c => c.MakeNavigation).Load();
context.Entry(sample).Collection(c => c.Drivers).Query().Load();
```

#### Ленивая загрузка

Представляет собой заргузку по требованию, когда навигационное свойство применяется для доступа к связанной записи, которая пока еще не загружена в память. По умолчанию отключена в EF Core.

Для включения ленивой загрузки навигационное свойство должно быть помечено ключевым словом virtual. Это свойство будет работать как прокси. Кроме этого нужно должным образом настроить контекст данных.

Необходимо установить пакет:

```csharp
dotnet add package Microsoft.EntityFrameworkCore.Proxies
```

Включить прокси в конфигрурации:

```csharp
optionsBuilder.UseLazyLoadingProxies();
```

Пометить в классах сущностей все навигационные свойства как виртуальные:

```csharp
public class Sample : BaseEntity
{
    ...
    public int MakeId { get; set; }
    [ForeignKey(nameof(MakeId))]
    public virtual Make MakeNavigation { get; set; }
    public virtual Ratio RatioNavigation { get; set; }
    [InverseProperty(nameof(Driver.Samples))]
    public virtual IEnumerable<Driver> Drivers { get; set; } = new List<Driver>();
    [InverseProperty(nameof(SampleDriver.SampleNavigation))]
    public virtual IEnumerable<SampleDriver> SampleDrivers { get; set; } = new List<SampleDriver>();
}
```

Пример:

```csharp
using (var context = new ApplicationDbContext(new DbContextOptions<ApplicationDbContext>()))
{
    var sample = context.Samples.First(x => x.Id == 1);
    var make = sample.MakeNavigation; // здесь происходит дополнительная загрузка сущности
}
```