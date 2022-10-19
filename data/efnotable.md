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

## Проекцирование данных

Модели в EF Core могут быть получены через проецирование - преобразование данных в другой произвольный тип данных. 

Пример простого проецирования:

```csharp
var ids = context.Samples.Select(x => x.Id).ToList();
```

Более сложный пример:

```csharp
var svm = context.Samples.Select(x => new SampleViewModel
{
    Id = x.Id,
    Name = x.Name,
});
```

## Обработка генерируемых базой данных значений

Платформа EF Core способна после добавления или изменения сущности формировать в SQL запросах автоматически корректные новые значения в поля сущности.

Пример добавления:

```csharp
var sample = new Sample
{
    Name = "Test",
    MakeId = 1,
};
context.Samples.Add(sample);
context.SaveChanges();
```

Пример изменения:

```csharp
var sample = context.Samples.First(x => x.Id == 1);
sample.Name = "Test1";
context.SaveChanges();
```

## Конкуренктный доступ

Проблема конкуренктного доступа возникает, когда несколько процессов производят обновление одной и той-же самой записи одновременно. 

Многие базы данных имеют специальный тип данных (timestamp или rowversion) для облегчения решения этой проблемы. Поле с этой пометкой будет автоматически обновляться после каждого изменения записи. Тип данных TimeStamp из базы данных будет отображен на свойство сущности byte[] языка C#. Свойства должны быть декорированы атрибутом Timestamp или в Fluent API должна быть добавлена настройка обновления значения этого поля после обновления, добавления и удаления записи в таблице базы данных.

## Устойчивость подключения

Многие СУБД имеют встроенный механизм повторных попыток для устранения сбоев в системе базы данных, который может быть использовать платформа EF Core. При включении SqlServerRetryingExecutionStrategy платформа сама при возникновении ошибки автоматически повторяет операцию, пока не будет достигнуто максимальное количество попыток.

Для включения в SqlServer нужно использовать метод EnableRetryOnFailure().

Пример:

```csharp
optionsBuilder.UseSqlServer(connectionString, options => options.EnableRetryOnFailure());
```

Как только количество повторных подключений достигнуто, то EF Core уведомляет приложение о том, что возникла проблема с подключением путем броска исключения RetryLimitExceededException.

## Отображение хранимых функций

Функции базы данных могуть быть отражены на C#-методы и могут быть включены в LINQ-выражения. EF Core поддерживает множество встроенных функций SQL сервера. Оператор C# ?? отражается на функцию coalese SQL сервера. String.IsNullOrEmpty() отражается на функцию проверки на null и использует функцию len SQL сервера для проверки длинны строки.

Чтоб использовать эту функцию в C#, нужно добавить новый метод в унаследованный от DbContext класс.

Пример типичной функции базы данных SQL Server:

```sql
CREATE FUNCTION test_SamplesCountFor ( @makeId int )
RETURNS int
AS
BEGIN
    DECLARE @Result int
    SELECT @Result = COUNT(makeId) FROM dbo.Samples WHERE makeId = @makeId
    RETURN @Result
END
GO
```

Пример добавленного метода:

```csharp
public partial class ApplicationDbContext : DbContext
{
    [DbFunction("test_SamplesCountFor", Schema = "dbo")]
    public static int SamplesCountFor(int makeId) => throw new NotSupportedException();
    ...
}
```

Пример использования этого метода:

```csharp
var makes = context.Makes
    .Where(x => ApplicationDbContext.SamplesCountFor(x.Id) > 1).ToList();
```

Платформа EF Core поддерживает отображение функций, вертающих в качестве результата значение-таблицу.

Пример типичной функции базы данных SQL Server, возвращающей в качестве результата значение-таблицу:

```sql
CREATE FUNCTION test_GetSamplesForMake ( @makeId int )
RETURNS TABLE
AS
RETURN
(
    SELECT Id, IsTest, Name, MakeId
    FROM MySample WHERE MakeId = @makeId
)
GO
```

Пример добавленного метода:

```csharp
public partial class ApplicationDbContext : DbContext
{
    [DbFunction("test_GetSamplesForMake", Schema = "dbo")]
    public IQueryable<Sample> GetSamplesFor(int makeId) => FromExpression(() => GetSamplesFor(makeId));
    ...
}
```

Пример использования этого метода:

```csharp
var samples = context.GetSamplesFor(1).ToList();
```

## Функции EF.Functions

Статический класс EF предоставляет множество методов C#, отражающихся на специальные функции базы данных.

Наиболее полезные методы:

Метод | Описание
------|------
Like() | Реализация операции SQL LIKE. Для SQL Server сравнение регистронезависимое с предоставлением подстановочного символа %
Random() | Возвращает псевдослучайное число от 0 до 1, отражается на функцию RAND базы данных
Collation<TProperty>() | Задает параметры сопоставления, которые будут использоватся в запросе LINQ
Contains() | Отражается на функцию CONTAINS базы данных. Таблица должна быть проиндексирована в полнотекстовом поиске.
FreeText() | Отражается на функцию FREETEXT базы данных. Таблица должна быть проиндексирована в полнотекстовом поиске.
DataLength() | Количество байт, необходимых для предоставления выражения
DateDiffYear() ... DateDiffNansecond() | Отражается на функцию DATEDIFF базы данных. Возвращает интервал между двумя датами
DateFromParts() | Инициализирует структуру новую DateTime и отражается на функцию DATEFROMPARTS базы данных
DateTime2FromParts() | Инициализирует структуру новую DateTime и отражается на функцию DATETIME2FROMPARTS базы данных
DateTimeFromParts() | Инициализирует структуру новую DateTime и отражается на функцию DATETIMEFROMPARTS базы данных
DateTimeOffsetFromParts() | Инициализирует структуру новую DateTimeOffset и отражается на функцию DATETIMEOFFSETFROMPARTS базы данных
SmallDateTimeFromParts() | Инициализирует структуру новую DateTime и отражается на функцию SMALLDATETIMEFROMPARTS базы данных
TimeFromParts() | Инициализирует структуру новую TimeSpan и отражается на функцию TIMEFROMPARTS базы данных
IsDate() | Проверка что строка является датой. Отражается на функцию ISDATE базы данных

Пример:

```csharp
var samples = context.Samples.IgnoreQueryFilters().Where(x => EF.Functions.Like(x.Name, "%Test%")).ToList();
```

## Пакетирование операторов

В EF Core значительно повышена производительность при сохранении изменений в базе данных за счет выполнения операторов в одном и более пакетов. Исполняющая среда EF Core пакетирует операторы создания, обновления и удаления с использованием табличных параметров.

Когда, например, добавляется несколько записей в рамках одной транзакции, то исполняющая среда EF Core пакетирует операторы в одиночное сообщение.


973 (1022)

















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




