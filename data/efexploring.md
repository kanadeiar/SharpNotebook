# Работа EF

## Создание записей данных

Записи добавляются в базу данных после того, как они созданы в коде, добавлены через DbSet<T> и вызван метод SaveChanges(). В этом методе ChangeTracker сообщает, что сущности были добавлены и EF Core создает SQL инструкции добавления записей в базу данных.

### EntityState

Пока сущность, созданная в коде не добавлена через DbSet<T>, статус EntityState будет равен Detached. После того, как сущность будет добавлена, ее статус будет Added. После вызова метода SaveChanges(), статус будет равен Unchanged.

### Добавление записей

Добавить новую запись в базу данных можно путем создания нового объекта, добавления его через DbSet<T> методом Add() и вызова метода SaveChanges(). 

Пример:

```csharp
using (var context = new ApplicationDbContext(new DbContextOptions<ApplicationDbContext>()))
{
    var newSample = new Sample();
    context.Samples.Add();
    context.SaveChanges();
}
```

Можно использовать и метод Attach() для добавления новой записи.

Пример:

```csharp
using (var context = new ApplicationDbContext(new DbContextOptions<ApplicationDbContext>()))
{
    var newSample = new Sample();
    context.Samples.Attach(newSample);
    context.SaveChanges();
}
```

Можно добавлять сразу несколько записей в базу данных использованием метода AddRange().

Пример:

```csharp
using (var context = new ApplicationDbContext(new DbContextOptions<ApplicationDbContext>()))
{
    var list = new List<Sample>{
        new Sample(),
        new Sample(),
        new Sample(),
    };
    context.Samples.AddRange(list);
    context.SaveChanges();
}
```

Можно добавляь в базу данных объектные графы - несколько взаимосвязанных объектов, которые являются несколькими записями в разных таблицах базы данных. EF Core создаст связи автоматически.

Пример:

```csharp
using (var context = new ApplicationDbContext(new DbContextOptions<ApplicationDbContext>()))
{
    var make = new Make { Name = "Make" };
    var sample = new Sample { Name = "One" };
    ((List<Sample>)make.Samples).Add(sample);
    context.Makes.Add(make);
    context.SaveChanges();
}
```

Можно добавлять в базу данных в разные таблицы данных связанные записи типа "многие-ко-многим" в удобной и быстрой форме.

Пример:

```csharp
using (var context = new ApplicationDbContext(new DbContextOptions<ApplicationDbContext>()))
{
    var drivers = new List<Driver>
    {
        new() { PersonInfo = new Person { FirstName = "One", LastName = "Two" }},
        new() { PersonInfo = new Person { FirstName = "Three", LastName = "Four" }},
    };
    var samples = context.Samples.Take(2).ToList();
    ((List<Driver>)samples[0].Drivers).AddRange(drivers.Take(..1));
    ((List<Driver>)samples[1].Drivers).AddRange(drivers.Take(1..));
    context.SaveChanges();
}
```

Добавление множества записей в базу данных возможно вызовыми простых методов производного от DbContext класса.

Пример:

```csharp
using (var context = new ApplicationDbContext(new DbContextOptions<ApplicationDbContext>()))
{
    List<Make> makes = new()
    {
        new() { Name = "Test" },
        new() { Name = "Dott" },
    };
    context.Makes.AddRange(makes);
    context.SaveChanges();
    List<Sample> samples = new()
    {
        new() { MakeId = 1, Name = "Vova" },
        new() { MakeId = 2, Name = "Maka" },
    };
    context.Samples.AddRange(samples);
    context.SaveChanges();
}
```

### Столбец идентификации

Когда сущность имеет целочисленное свойство определенное как первичный ключ, то такое свойство будет отражатся как первичный ключ SQL Server. EF Core считает, что всякая сущность со значением ключа 0 является новой, а со значением каким-либо - уже существует в базе данных.

Когда нужно добавлять записи с определенными идентфикаторами, то нужно вызывать метод ExecuteSqlRaw() и параметрах метода передавать SQL код деактивации/активации IDENTITY_INSERT.

Пример:

```csharp
context.Database.ExecuteSqlRaw($"SET IDENTITY_INSERT {schema}.{tableName} ON");
//код добавления записей с идентификаторами
context.Database.ExecuteSqlRaw($"SET IDENTITY_INSERT {schema}.{tableName} OFF");
```

## Очистка данных

Можно быстро уалить все записи из базы данных с использованием специальных команд. Используется метод FindEntityType() на свойстве модели для получения таблицы и названия схемы для дальнейшего удаления. После удаления записи нужно использовать команду "DBCC CHECKIDENT" для сброса идентификатора каждой таблицы.

Пример:

```csharp
using (var context = new ApplicationDbContext(new DbContextOptions<ApplicationDbContext>()))
{
    var entities = new[]
    {
        typeof(Driver).FullName,
        typeof(Sample).FullName,
        typeof(Make).FullName,
    };
    foreach (var entityName in entities)
    {
        var entity = context.Model.FindEntityType(entityName);
        var tableName = entity.GetTableName;
        var schemaName = entity.GetSchema;
        context.Database.ExecuteSqlRaw($"DELETE FROM {schemaName}.{tableName}");
        context.Database.ExecuteSqlRaw($"DBCC CHECKIDENT (\"{schemaName}.{tableName}\", RESEED, 0)");
    }
}
```

## Получение записей данных

Можно применять метод ToQueryString() для получения текста SQL запроса, который будет исполнятся платформой EF Core.

Для получения всех записей в простой манере нужно использовать свойство DbSet<T> напрямую без использования LINQ конструкций и далее для немедленного получения результатов добавить метод ToList(). 

Пример:

```csharp
using (var context = new ApplicationDbContext(new DbContextOptions<ApplicationDbContext>()))
{
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
}
```

### Фильтрация

Для отфильтровывания получаемых из базы данных записей следует применять метод Where(). Множественное примененение методов Where() друг за другом допускается.

Пример:

```csharp
using (var context = new ApplicationDbContext(new DbContextOptions<ApplicationDbContext>()))
{
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
}
```

### Сортировка

Для сортировки получаемых из базы данных значений следует применять методы OrderBy(), OrderByDescending(). Если нужна дополнительная сортировка подэлементов, то применять нужно метод ThenBy() и ThenByDescending().

Пример:

```csharp
using (var context = new ApplicationDbContext(new DbContextOptions<ApplicationDbContext>()))
{
    var samples = context.Samples.OrderBy(x => x.Name).ThenByDescending(x => x.AdvancedName);
    foreach (var e in samples)
    {
        Console.WriteLine($"{e.Name} {e.AdvancedName}");
    }
}
```

Для сортировки в обратном порядке всех элементов можно применять метод Reverse().

Пример:

```csharp
using (var context = new ApplicationDbContext(new DbContextOptions<ApplicationDbContext>()))
{
    var samples = context.Samples.OrderBy(x => x.Name).ThenBy(x => x.AdvancedName).Reverse();
    foreach (var e in samples)
    {
        Console.WriteLine($"{e.Name} {e.AdvancedName}");
    }
}
```

### Пагинация

Можно разбивать получаемые данные постранично - производить пагинацию данных с помощью методов Skip() и Take().

Пример:

```csharp
using (var context = new ApplicationDbContext(new DbContextOptions<ApplicationDbContext>()))
{
    var samples = context.Samples.Skip(2).Take(3).ToList();
    foreach (var e in samples)
    {
        Console.WriteLine($"{e.Name} {e.AdvancedName}");
    }
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
using (var context = new ApplicationDbContext(new DbContextOptions<ApplicationDbContext>()))
{
    var one = context.Samples.Where(x => x.Id == 1).First();
    var two = context.Samples.FirstOrDefault(x => x.Id == 1);
    var third = context.Samples.Find(12);
    context.ChangeTracker.Clear();
}
```

### Агрегация данных

EF Core поддерживает методы агрегации данных, выполняемых на серверной стороне. Это такие методы как Max(), Min(), Count(), Average(). 

Пример:

```csharp
using (var context = new ApplicationDbContext(new DbContextOptions<ApplicationDbContext>()))
{
    var count = context.Samples.Count(x => x.IsTest);
    var one = context.Samples.Max(x => x.Id);
    var two = context.Samples.Average(x => x.Id);
}
```

### Получение результата от записей по критерию

С помощью методов Any() и All() можно получить информацию о результате нахождения хотябы одной записи по критерию (Any) или все записи соответствуют критерию (All).

Пример:

```csharp
using (var context = new ApplicationDbContext(new DbContextOptions<ApplicationDbContext>()))
{
    var yes = context.Samples.Any(x => x.IsTest);
    var no = context.Samples.All(x => x.Id == 1);
}
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
using (var context = new ApplicationDbContext(new DbContextOptions<ApplicationDbContext>()))
{
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
}
```

## Связанные данные



994
















