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




















### Принадлежащие сущностные классы

В EF Core допустимо применение класса в качестве свойства сущности с целью создания коллекции свойств для другой сущности. Инфраструктура сама добавит все свойства из сущностного класса [Owned] к владеющему классу. Дает возможность многократного использования кода. 

Под капотом EF Core считает это отношением "один-к-одному". Владеющий класс - главная сущность. 

Пример:

```csharp
[Owned]
public class Person
{
    [Required, StringLength(100)]
    public string FirstName { get; set; }
    [Required, StringLength(100)]
    public string LastName { get; set; }
}
public class Driver : BaseEntity
{
    public Person PersonInfo { get; set; } = new Person();
    ...
}
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Driver>(x =>
    {
        x.OwnsOne(o => o.PersonInfo, pd =>
        {
            pd.Property<string>(nameof(Person.FirstName))
                .HasColumnName(nameof(Person.FirstName))
                .HasColumnType("nvarchar(50)");
            pd.Property<string>(nameof(Person.LastName))
                .HasColumnName(nameof(Person.LastName))
                .HasColumnName("nvarchar(50)");
        });
        x.Navigation(d => d.PersonInfo).IsRequired(true);
    });
    ...
```

Ограничения при работе с принадлежащими сущностными классами:

- Нельзя создавать свойства DbSet<T> с принадлежащими сущностными типами

- Нельзя делать вызов Entity<T>() в собственном типе в построителе модели

- Экземпляры принадлежащих сущностных классов не могут быть использованы в нескольких владельцах

- Принадлежащие сущностные типы не поддерживают наследование


1012




