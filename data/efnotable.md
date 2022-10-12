# Фичи EF

## Глобальные фильтры запросов

Глобальные фильтры запросов позволяют добавлять конструкцию where во все запросы LINQ для определенной сущности. Инфраструктура EF Core позволяет добавлять к сущности глобальный фильтр запросов, который затем применяется к каждому запросу, вовлекающему эту сущность.

Пример добавления глобального фильтра запросов к сущности:

```csharp
public void Configure(EntityTypeBuilder<Sample> builder)
{
    builder.HasQueryFilter(x => x.IsTest == true);
    ...
}
```

Примеры запросов сущностей из базы данных:

```csharp
var filtereds = context.Samples.ToArray(); // только со значением IsTest = true
var alls = context.Samples.IgnoreQueryFilters().ToArray(); // игнор фильтра
```

Глобальные фильтры запросов можно устанавливать на навигационных свойствах. 

Пример добавления глобального фильтра на навигационное свойство:

```csharp
modelBuilder.Entity<Ratio>().HasQueryFilter(e =>
    e.SampleNavigation.IsTest);
```

Примеры запросов сущностей из базы данных:

```csharp
var filtereds = context.Ratios.ToArray(); // отфильтрованные сущности
var alls = context.Samples.IgnoreQueryFilters().ToArray(); // игнор фильтра
```

Настроенный глобальный фильтр запросов на сущности работает также и в случае явной заргрузки связанных сущностей.

Пример запросов:

```csharp
var make = context.Makes.First(x => x.Id == 1);
context.Entry(make).Collection(x => x.Samples).Load(); // загрука сущностей отфильтрованных
var samples = make.Samples.ToArray();
context.Entry(make).Collection(x => x.Samples).Query().IgnoreQueryFilters().Load(); // игнор фильра
var samples2 = make.Samples.ToArray();
```

## Выполнение низкоуровневых запросов SQL

Инфраструктура EF Core располагает механизмом, позволяющим выполнять низкоуровневые операторы SQL в DbSet<T>. Методы FromSqlRaw() и FromSqlRawInterpolated() принимают строку, которая становится основой запроса LINQ. Такой запрос выполняется на стороне сервера.

В случае использования переменных нужно использовать версию с интерполяцией строк для обеспечения дополнительной защиты.

Пример:

```csharp
var sampleId = 1;
var sample = context.Samples
    .FromSqlInterpolated($"Select * from dbo.MySample where Id = {sampleId}")
    .Include(x => x.MakeNavigation)
    .First();
```

Нужно знать правила, которые следует соблюдать в случае применения низкоуровневых запросов SQL:

- запрос SQL должен возвращать данные для всех свойств сущностного типа;

- имена столбцов должны совпадать с именами свойств, с которыми они сопоставляются;

- запрос SQL не пожет содержать связанные данные.



958




