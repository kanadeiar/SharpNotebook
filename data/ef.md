# Основы EF

Общая цель EF — предоставить возможность взаимодействия с данными из реляционных баз данных с использованием объектной модели, которая отображается напрямую на бизнес-объекты (или объекты предметной области) в создаваемом приложении.

Инфраструктура ADO.NET снабжает структурой, которая позволяет выбирать, вставлять, обновлять и удалять данные с помощью объектов подключений, команд и чтения данных.

Инфраструктура EF сокращает разрыв между целями и оптимизацией базы данных и целями и оптимизацией объектно-ориентированного программирования. С помощью EF можно работать с базами данных, не сталкиваясь со строками кода SQL. Выполняемая среда EF самостоятельно генерирует подходящие операторы SQL.

## Составные части EF

### Класс DbContext

Класс DbContext входит в состав главных компонентов EF Core и предоставляет доступ к базе данных через свойство Database. Объект DbContext управляет экземпляром ChangeTracker, поддерживает виртуальный метод OnModelCreating () для доступа к текучему API-интерфейсу (Fluent API), хранит все свойства DbSet<T> и предлагает метод SaveChanges ( ) , позволяющий сохранять данные в хранилище. Он применяется не напрямую, а через специальный класс, унаследованный от DbContext. Именно в этом классе размещены все свойства типа DbSet <T>.

Класс DbContext предоставляет основную функциональность во время работы со средством CodeFirst инфраструктуры EF. Это не означает невозможность примеения EF с существующей базой данных. Можно как использовать Code First с существующей базой данных или создать новую базу данных на основе сущностей с помощью миграций EF.

Команды:

```csharp
dotnet add package Microsoft.EntityFrameworkCore
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
```

Пример производного класса от DbContext специфичной предметной области, с переданной строкой подключения:

```csharp
public class ApplicationDbContext : DbContext
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options) : base(options)
    {
    }
}
```

Некоторые члены DbContext:

Член                     | Описание
-------------------------|-------------------------
Database         | Доступ к функциональности базы данных
Model            | Метаданные о сущности, об их отношениях и как они отображаются на базу данных
ChangeTracker    | Предоставляет доступ к информации и операциям для отслеживаемых экземпляров сущностей
DbSet<T>         | Не член DbContext, а свойства, добавляемые в пользовательский производный класс. Свойства используются для добавления, удаления и обновления сущностей, а запросы LINQ преобразуются в запросы SQL
Entry()          | Предоставляет доступ к опреациям и информации по сущностям или изменение состояния изменением EntityState
Set<TEntity>()   | Создает экземпляр свойства, используемый для запроса и сохранения данных
SaveChanges()/SaveChangesAsync() | Сохраняет изменения сущностей в базе данных и возвращает количество затронутых записей.
Add()/AddRange() Update()/UpdateRange() Remove()/RemoveRange() | Методы манипулирования сущностями. 
Find()           | Находит сущность с заданным значением первичного ключа
Attach()/AttachRange() | Ставит на отслеживание сущности
OnModelCreating() | Вызывается при инициализации модели, но до ее завершения. Методы из Fluent API помещаются в этот метод.
OnConfiguring()  | Построитель, используемый для создания или изменения параметров DbContext. Выполняется каждый раз при создании экземпляра DbContext.

### Конфигурирование

Экземпляр DbContext конфигурируется с использованием экземпляра класc а DbContextOptions. ЭкземплярDbContextOptions создается с применением DbContextOptionsBuilder. Через экземпляр  dbContextOptionsBuilder выбирается поставщик базы данных (наряду с любыми настройками, касающимися поставщика) и устанавливаются общие параметры экземпляра DbContext инфраструктуры EF Core (наподобие ведения журнала). Затем свойство Options внедряется в базовый класс DbContext во время выполнения.

### Фабрика DbContext этапа проектирования

Фабрика DbContext этапа проектирования представляет собой класс, который реализует интерфейс IDesignTimeDbContextFactory<T>, где Т — класс, производный от DbContext.

Пример:

```csharp
public class ApplicationDbContextFactory : IDesignTimeDbContextFactory<ApplicationDbContext>
{
    public ApplicationDbContext CreateDbContext(string[] args)
    {
        var optionsBuilder = new DbContextOptionsBuilder<ApplicationDbContext>();
        var connectionstring = @"server=.,5433;Database=AutoLotSamples;
User Id=sa;Password=P@sswOrd";
        optionsBuilder.UseSqlServer(connectionstring);
        Console.WriteLine(connectionstring);
        return new ApplicationDbContext(optionsBuilder.Options);
    }
}
```

### Сохранение изменений

Чтобы заставить DbContext и ChangeTracker сохранить любые изменения, внесенные в отслеживаемые сущности, нужно вызовать метод SaveChanges(). 

Исполняющая среда EF Core помещает каждый вызов SaveChanges()/SaveChangesAsync ( ) внутрь неявной транзакции, использующей уровень изоляции базы данных.

Пример явного применения транзакций:

```csharp
using var trans = context.Database.BeginTransaction();
try
{
    // Создать, изменить, удалить запись,
    context.SaveChanges();
    trans.Commit();
}
catch (Exception ex)
{
    trans.Rollback();
}
```

Можно применять точки сохранения:

```csharp
using var trans = context.Database.BeginTransaction();
try
{
    // Создать, изменить, удалить запись,
    trans.CreateSavepoint("check point 1");
    context.SaveChanges();
    trans.Commit();
}
catch (Exception ex)
{
    trans.RollbackToSavepoint("check point 1");
}
```

### Свойство типа DBSet<T>

Для каждой таблицы в объектной модели нужно предусмотреть свойство типа DbSet<T>. Класс DbSet<T> представляет собой специализированную коллекцию, используемую для взаимодействия с поставщиком баз данных с целью получения, добавления, обновления и удаления записей в базе данных. Для перевода свойств в режим ленивой загрузки эти свойства должны быть виртуальными.
    
Некоторые члены DBSet<T>:

Член                     | Описание
-------------------------|-------------------------
Add() AddRange()         | Позволяют вставлять в коллекцию новый объект (или пачку). Они получают статус EntityState.Added и будут добавлены в базу от SaveChanges() на DbContext
AsAsyncEnumerable()      | Возвращает коллекцию как реализацию IAsyncEnumerable<T>
AsQueryable()            | Возвращает экземпляр реализации lQueryable<T> из DbSet<T>
Attach() AttachRange()   | Ассоциирует объект с DbContext.
Find() FindAsync()       | Находит строку данных по первичному ключу и возвращает объект, представляющий эту строку.
Update() AddRange()      | Позволяют вставлять в коллекцию новый объект (или пачку). Они получают статус 
Remove() RemoveRange()   | Помечает объект - или группу - для последующего удаления
FromSqlRaw() FromSqlInterpolated() | Создает низкоуровневый запрос SQL, возвращяющий сущности в этом наборе

Все EF помещают свои вызовы SaveChanges()/SaveChangesAsync() внутрь транзакции. Внутрь неявной транзакции помещается и ExecuteSqlCommand().

## Экземпляр ChangeTracker

Экземпляр ChangeTracker отслеживает состояние объектов, загруженных в DbSet<T> внутри экземпляра DbContext.

Значения перечисления EntityState:

Значение                 | Описание
-------------------------|-------------------------
Detached                 | Объект есть, но не отслеживается. Сущность после создания и перед добавление ее к объектному контексту.
Unchanged                | Объект не изменился с момента присоединения к контексту, или с последнего SaveChanges()
Added                    | Объект - новый и добавлен в объектный контекст, метод SaveChanges() не вызывался
Deleted                  | Объект удален из контекста, но еще не удален из хранилища данных
Modified                 | Одно из скалярных свойств было изменено, но метод SaveChanges() не вызывался

Проверка состояния объекта:

```csharp
EntityState state = context.Entry(entity).State;
```

Установка состояния:

```csharp
context.Entry(entity).State = EntityState.Deleted;
```

### События ChangeTracker

Экземпляр ChangeTracker способен генерировать два события: StateChanged и Tracked. StateChanged - изменение состояния. Tracked - когда сущность начинает отслеживаться.

### Сброс состояния DbContext

Существует возможность возвращать производный DbContext в исходное состояние. Метод ChangeTracker.Clear() сбрасывает отлеживаение всех объектов в коллекции DbSet<T>, устанавливая их состояние в состояние Detached. Это увеличивает производительность.

## Сущности

Коллекции сущностей в приложении образуют модель физической базы данных, такая модель называется модель сущностных данных (EDM). Модель сопоставляется с предметной областью приложения. А сами сущности их их свойства отображаются на таблиы и столбцы базы данных. Образуется слабая связанность между базой данных и сущностями.

### Сопоставление свойств со столбцами

По соглашениям EF все открытые свойства, допускающие чтение и запись, сопоставляются со столбцами таблицы, на которую отражается сущность. 

### Сопоставление классов с таблицами

Существует два типа сопоставления: таблица на иерархию (TPH) и таблица на тип (TPT).

В таблице на иерархию (TPH) когда EF используется для создания таблицы в базе данных, то унаследованный класс объединяется с производным классом и создается единственная таблица.

Пример TPH:

```csharp
public abstract class BaseEntity
{
    public int Id { get; set; }
    public byte[] TimeStamp { get; set; }
}
public class Sample : BaseEntity
{
    public string Name { get; set; }
    public string Color { get; set; }
}
public partial class ApplicationDbContext : DbContext
{
    public DbSet<Sample> Samples => Set<Sample>();
    ...
```

В таблице на тип (TPT) когда EF используетя для создания таблицы в базе данных, то создаются таблицы на каждый класс, и они имеют отношение "один к одному". Для применения TPT необходимо проинструктировать для отображения каждого на таблицу с помощью аннотаций данных FluentAPI.

Пример настройки TPT (добавить в DBContext):

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<BaseEntity>().ToTable("BaseEntities");
    modelBuilder.Entity<Sample>().ToTable("Samples");
    OnModelCreatingPartial(modelBuilder);
}
```

## Навигационные свойства

Навигационные свойства позволяют находить связанные данные в других сущностях без необходимости в написании запросов JOIN. EF учитывает отношения между таблицами SQL Server как внешние ключи.

В EF каждый класс в модели содержит виртуальные свойства, которые соединяют вместе классы. EF поддерживает типы различные типы отношений: "один ко многим", "один к одному", "многие ко многим".

По соглашению, когда EF находит свойство по имени <Класс>.Id, оно будет применяться в качестве внешнего ключа для навигационного свойства <Класс>.

Если сущность с навигационным свойством типа ссылки не имеет свойства для значения внешнего ключа, тогда EF Core создаст необходимое свойство или свойства внутри сущности, известные как теневые свойства внешних ключей.

### Отношение один-ко-многим

Чтобы создать отношение "один ко многим", сущностный класс со стороны "один" (главная сущность) добавляет свойство типа коллекции сущностных классов, находящихся на стороне “многие” (зависимые сущности). Зависимая сущность также должна иметь свойства для внешнего ключа обратно к главной сущности.

Пример навигационных свойств в сущностях:

```csharp
public class Make : BaseEntity // Главная сущность
{
    public string Name { get; set; }
    public IEnumerable<Sample> Samples { get; set; } = new List<Sample>();
}
public class Sample : BaseEntity // Зависимая сущность
{
    public string Name { get; set; }
    public int MakeId { get; set; }
    public Make MakeNavigation { get; set; }
}
```

Пример FluentAPI:

```csharp
x.HasOne(s => s.MakeNavigation)
    .WithMany(p => p.Samples)
    .HasForeignKey(s => s.MakeId)
    .OnDelete(DeleteBehavior.ClientSetNull)
    .HasConstraintName("FK_Sample_Make_MakeId");
```

### Отношение один-к-одному

В отношениях "один к одному" обе сущности имеют навигационные свойства типа ссылок друг на друга. Хотя отношения "один к одному" четко обозначают главную и зависимую сущности, при построении таких отношений EF Core необходимо сообщить о том, какая сторона является главной, либо явно определяя внешний ключ, либо указывая главную сущность с использованием Fluent API.

Пример навигационных свойств в сущностях:

```csharp
public class Sample : BaseEntity // Главная сущность
{
    public string Name { get; set; }
    public Ratio RatioNavigation { get; set; }
}
public class Ratio : BaseEntity // Зависимая сущность
{
    public string RatioId { get; set; }
    public int SampleId { get; set; }
    public Sample SampleNavigation { get; set; }
}
```

Пример Fluent API:

```csharp
modelBuilder.Entity<Ratio>(x =>
{
    x.HasIndex(x => x.SampleId, "XII_Ratios_CarId")
        .IsUnique();
    x.HasOne(x => x.SampleNavigation)
        .WithOne(x => x.RatioNavigation)
        .HasForeignKey<Ratio>(x => x.SampleId);
});
```

### Отношение многие-ко-многим

В отношении "многие ко многим" каждая сущность содержит навигационное свойство типа коллекции для другой сущности, что в хранилище данных реализуется с использованием таблицы соединения посреди двух сущностных таблиц. Такая таблица соединения именуется в соответствии с двумя таблицами в виде <Сущность1Сущность2>. Таблица соединения имеет отношения "один ко многим" с каждой сущностной таблицей.

Пример навигационных свойств в сущностях:

```csharp
public class Sample : BaseEntity
{
    public string Name { get; set; }
    public IEnumerable<Driver> Drivers { get; set; } = new List<Driver>();
}
public class Driver : BaseEntity
{
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public IEnumerable<Sample> Samples { get; set; } = new List<Sample>();
}
```

Пример FluentAPI:

```csharp
modelBuilder.Entity<Sample>()
    .HasMany(x => x.Drivers)
    .WithMany(x => x.Samples)
    .UsingEntity<Dictionary<string, object>>(
        "SampleDriver",
        j => j.HasOne<Driver>()
        .WithMany()
        .HasForeignKey("DriverId")
        .OnDelete(DeleteBehavior.Cascade),
        j => j.HasOne<Sample>()
        .WithMany()
        .HasForeignKey("SamleId")
        .OnDelete(DeleteBehavior.ClientCascade)
    );
```

### Каскадное поведение

В большинстве хранилищ данных (вроде SQL Server) установлены правила, управляющие поведением при удалении строки. Если связанные (зависимые) записи тоже должны быть удалены , то такой подход называется каскадным удалением.

Есть три различные действия с сущностями при их удалении:

- зависимые записи удаляются;

- зависимые внешние ключи устанавливаются в null;

- зависимые сущности остаются незатронутыми.

Стандартное поведение для необязательных и обязательных отношений отличается. Поведение конфигурируется с применением перечисления DeleteBehavior посредством Fluent API. Указанное поведение в EF Core инициируется только после удаления сущности и вызова метода SaveChanges () на экземпляре класса, унаследованного от DbContext.

### Необязательные отношения

Необязательными отношениями считаются такие, в которых зависимая сущность может устанавливать значение или значения внешних ключей в null. Для необязательных отношений стандартным поведением является ClientSetNull.

### Обязательные отношения

Обязательные отношения — это такие отношения, при которых зависимая сущность не может устанавливать значение или значения внешних ключей в null. Для обязательных отношений стандартным поведением является Cascade.

## Соглашения, связанные с сущностями

Соглашения всегда включены, если только они не отменены аннотациями данных или кодом Fluent API.

Важные соглашения EF Core:

Соглашение | Описание
-----------|----------
Включаемые таблицы | В базе данных создаются все классы со свойством DbSet и все классы, которые достижимы (через навигационные свойства) классом DbSet
Включаемые столбцы | На столбцы отображаются все открытые свойства с методами получения и установки (включая автоматические свойства)
Имя таблицы | Отображается на имя свойства DbSet в классе, производном от DbContext. Если свойств DbSet не существует, тогда используется имя класса
Схема | Таблицы создаются в стандартной схеме хранилища данных (dbo в SQL Server)
Имя столбца | Имена столбцов отображаются на имена свойств класса
Тип данных столбца | Типы данных выбираются на основе типов данных .NET и транслируются поставщиком базы данных (SQL Server). Тип данных DateTime отображается на datetime2(7), a string — на nvarchar(max). Строки, являющиеся частью первичного ключа, отображаются на nvarchar(450)
Допустимость значения null в столбце | Типы данных, не допускающие значение null, создаются как постоянные
столбцы Not Null.
Первичный ключ | Свойства с именами Id или <ИмяСущностногоТипа>Id будут конфигурироваться как первичный ключ. Ключи типа short, int, long или Guid имеют значения, управляемые хранилищем данных. Числовые значения создаются в виде столбцов Identity (SQLServer)
Отношения | Отношения между таблицами создаются при наличии навигационных свойств между сущностными классами
Внешний ключ | Свойства с именами <ИмяДругогоКласса>Id являются внешними ключами для навигационных свойств типа <ИмяДругогоКласса>

### Отображение свойств на таблицы

По соглашению открытые свойства для чтения и записи отображаются на столбцы с теми же самыми именами. Типы данных столбцов соответствуют эквивалентам для типов данных CLR свойств, принятым в хранилище данных. Свойства, не допускающие null, устанавливаются в хранилище данных как не null, а свойства, допускающие null, устанавливаются так, чтобы значение null было разрешено. Инфраструктура EF Core поддерживает ссылочные типы, допускающие null.

### Переопределение соглашений

Соглашения могут быть переопределены использованием метода ConfigureConventions() в классе DbContext. 

Пример:

```csharp
protected override void ConfigureConventions(ModelConfigurationBuilder configurationBuilder)
{
    configurationBuilder.Properties<string>().HaveMaxLength(100);
}
```

## Аннотации данных Entity Framework

Аннотации данных — это атрибуты С#, которые применяются для дальнейшего придания формы сущностям. Аннотации данных переопределяют любые конфликтующие соглашения.

Часто используемые аннотации данных:

Аннотация данных | Описание
-----------|----------
Table | Определяет имя схемы и таблицы для сущности
EntityTypeConfiguration | Поддержка настройки собственного класса использованием Fluent API
Keyless | Сущность не имеет ключа
Column | Определение имени столбца
BackingField | Указывает поддерживающее поле для свойства
Key | Определяет первичный ключ для сущности
Index | Указание в классе индекса из одного или нескольких столбцов
Owned | Объявление, что классом будет владеть другой сущностный класс
Required | Свойство не допускающее значения null в базе данных
ForeignKey | Свойство, применяемое в качестве внешнего ключа для навигационного свойства
InverseProperty | Навигационное свойство на другом конце отношения
StringLength | Максимальная длинна для свтрокового свойства
TimeStamp | Тип как rowevrsion и добавляет проверки параллелизма к операциям базы данных
ConcurrencyCheck | Поле флагов, предназначенное для использования во время проверки параллелизма при выполнении обновлений и удалений
DatabaseGenerated | Указывает на генерацию значания базой данных
DataType | Более специфичное определение поля, чем внутренний тип данных
Unicode | Отображает строковое свойство на колонку базы данных определенной кодировки без специфичности типа
Precision | Уточнение точности и масштабирования для колонки базы данных без уточнения типа
NotMapped | Исключает свойство или класс из полей или таблиц базы данных

Пример применения аннотаций данных:

```csharp
public abstract class BaseEntity
{
    [Key, DatabaseGenerated(DatabaseGeneratedOption.Identity)]
    public int Id { get; set; }
    [Timestamp]
    public byte[] TimeStamp { get; set; }
}
[Table("MySample", Schema = "dbo")]
[Index(nameof(MakeId), Name = "IX_MySample_MakeId")]
public class Sample : BaseEntity
{
    [Required, StringLength(40)]
    public string Name { get; set; }
    public int MakeId { get; set; }
    [ForeignKey(nameof(MakeId))]
    public Make MakeNavigation { get; set; }
    public Ratio RatioNavigation { get; set; }
    [InverseProperty(nameof(Driver.Samples))]
    public IEnumerable<Driver> Drivers { get; set; } = new List<Driver>();
}
```

## Fluent API

С помощью интерфейса Fluent API сущности приложения конфигурируются посредством кода C#. Методы предоставляются объектом ModelBuilder в методе OnModelCreating() в классе DbContext. Является самым мощным способом конфигурирования и переопределяет любые конфликтующие между собой соглашения и аннотации данных.

Пример настройки:

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Sample>(x =>
    {
        x.ToTable("MySamples", "dbo");
        x.HasKey(x => x.Id);
        x.HasIndex(x => x.MakeId, "IX_Index_1");
        x.Property(x => x.Name)
            .IsRequired()
            .HasMaxLength(50);
        x.Property(x => x.TimeStamp)
            .IsRowVersion()
            .IsConcurrencyToken();
    });
...
}
```

### Стандартные значения

Пример установки стандартного значения:

```csharp
x.Property(x => x.Name)
    .IsRequired()
    .HasMaxLength(50)
    .HasDefaultValue("Копейка");
```

### Поддерживающее поле

Для информаирования EF Core о поддерживающем поле для определенного свойства сущности используется средство Fluent API:

```csharp
private bool? _isTest;
public bool IsTest
{
    get => _isTest ?? true;
    set => _isTest = value;
}
x.Property(x => x.IsTest)
    .HasField("_isTest")
    .HasDefaultValue(true);
```

### Оптимизированные столбцы

Можно оптимизировать столбцы, хранящие значения null. Нужно использовать метод IsSparce().

Пример:

```csharp
x.Property(x => x.IsTest)
    .IsSparse();
```

### Вычисляемые столбцы

Столбцы могут вычисляться на основе возможностей хранилища данных. Плюс есть возможность сохранения вычисляемых данных, чтобы их значение каждый раз не вычислялось при запросе данных.

Пример:

```csharp
x.Property(x => x.AdvancedName)
    .HasComputedColumnSql("Advanced [Name]", stored: true);
```

### Проверка условий

Можно описывать проверку входных данных на каждую строку. При выполнении условий - выполнять определенное действие.

Пример:

```csharp
modelBuilder.Entity<Make>()
    .HasCheckConstraint(name: "CH_Name", sql: "[Name]<>'Test'", buildAction: c => c.HasName("CK_Check_Name"));
```

### Исключение сущностей из миграций

Можно удалить одну сущность из системы миграций в DbContext в методе OnModelCreating().

Пример:

```csharp
modelBuilder.Entity<BaseEntity>()
    .ToTable("BaseEntities", t => t.ExcludeFromMigrations());
```

### Использование классов Fluent API для сущностей

Можно в EF Core выносить инструкции Fluent API в отдельный класс с настройками для каждой отдельной сущности. 

Пример вынесенного в папку "Configuration" класса SampleConfiguration:

```csharp
public class SampleConfiguration : IEntityTypeConfiguration<Sample>
{
    public void Configure(EntityTypeBuilder<Sample> builder)
    {
        builder.ToTable("MySamples", "dbo");
        builder.HasKey(x => x.Id);
        builder.HasIndex(x => x.MakeId, "IX_Index_1");
        builder.Property(x => x.Name)
            .IsRequired()
            .HasMaxLength(50)
            .HasDefaultValue("Копейка");
        builder.Property(x => x.TimeStamp)
            .IsRowVersion()
            .IsConcurrencyToken();
        builder.Property(x => x.IsTest)
            .HasField("_isTest")
            .HasDefaultValue(true);
        builder.Property(x => x.IsTest)
            .IsSparse();
        builder.Property(x => x.AdvancedName)
            .HasComputedColumnSql("Advanced [Name]", stored: true);
        builder.HasOne(s => s.MakeNavigation)
            .WithMany(p => p.Samples)
            .HasForeignKey(s => s.MakeId)
            .OnDelete(DeleteBehavior.ClientSetNull)
            .HasConstraintName("FK_Sample_Make_MakeId");
        builder
            .HasMany(x => x.Drivers)
            .WithMany(x => x.Samples)
            .UsingEntity<SampleDriver>(
                j => j.HasOne(x => x.DriverNavigation)
                     .WithMany(d => d.SampleDrivers)
                     .HasForeignKey(nameof(SampleDriver.DriverId))
                     .OnDelete(DeleteBehavior.Cascade),
                j => j.HasOne(x => x.SampleNavigation)
                     .WithMany(d => d.SampleDrivers)
                     .HasForeignKey(nameof(SampleDriver.SampleId))
                     .OnDelete(DeleteBehavior.ClientCascade),
                j =>
                {
                    j.HasKey(x => new { x.SampleId, x.DriverId });
                }
            );
    }
}
// Добавить использование класса конфигурации в метод создания моделей в DBContext
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    new SampleConfiguration().Configure(modelBuilder.Entity<Sample>());
    ...
}
// Добавить аттрибут в класс сущности, уточняющий класс конфигурирования сущности
...
[EntityTypeConfiguration(typeof(SampleConfiguration))]
public class Sample : BaseEntity
{
    ...
}
```

## Типы запросов

Типы запросов — это коллекции DbSet<T>, которые применяются для изображения представлений, оператора SQL или таблиц без первичного ключа. Типы запросов добавляются к производному классу DbContext с применением свойств DbSet<T> и конфигурируются как не имеющие ключей.

Требуемый класс SampleViewModel конфигурируется с атрибутом [Keyless].

```csharp
[Keyless]
[EntityTypeConfiguration(typeof(SampleViewModelConfiguration))]
public class SampleViewModel
{
    public string Name { get; set; }
    [NotMapped]
    public string FullDetail => $"The {Name}";
    public override string ToString() => FullDetail;
}
// в DbContext
new SampleViewModelConfiguration().Configure(modelBuilder.Entity<SampleViewModel>());
// конфигурация
namespace SimpleConsole.ViewModels.Configuration;

public class SampleViewModelConfiguration : IEntityTypeConfiguration<SampleViewModel>
{
    public void Configure(EntityTypeBuilder<SampleViewModel> builder)
    {
        builder.HasNoKey().ToView("SampleMakeView", "dbo");
        // builder.HasNoKey().ToSqlQuery(@" // запрос SQL
        // SELECT s.Name Name, m.Name MakeName
        // FROM dbo.Samples s
        // INNER JOIN dbo.Make m ON s.MakeId = m.Id").ToView("SampleMakeView");        
        builder.ToTable(x => x.ExcludeFromMigrations());
    }
}
```

### Гибкое сопоставление с запросом или таблицей

Есть возможность сопоставления одного и того же класса с более чем одним объектом базы данных. Такими объектами могут быть таблицы, представления или функции.

Пример:

```csharp
modelBuilder.Entity<SampleViewModel>()
    .ToTable("Samples")
    .ToView("SampleWithMakeView");
```

## Выполнение запросов

Запросы на извлечение данных создаются использованием запросов LINQ в отношении свойств DbSet<>. На стороне сервера запросы LINQ превращаются в инструкции SQL. Запросы не выполняются до тех пор, пока не начнется проход по результатам запросов.

Пример отложенного запроса и немедленного:

```csharp
var samples = context.Samples.Where(x => x.Name == "Test"); // оценка
var result = samples.ToList(); // выполнение
result = context.Samples.Where(x => x.Name == "Sim").ToList(); // немедленное выполнение
var query = context.Samples.AsQueryable(); // запрос
query = query.Where(x => x.Name == "Two");
result = query.ToList(); // получение
```

В EF Core возможность смешанного выполнения была изменена и теперь выполнять на стороне клиента можно только последний узел оператора LINQ.

### Отслеживаемые и неотслеживаемые запросы

По умолчанию сущности при получении через DbSet<T> отслеживаются компонентом ChangeTracker. Любые последующие обращения к базе данных за тем же самым элементом на основе перивичного ключа будут приводить к обновлению элемента, а не к его дублированию. Это снижает производительность.

Чтоб сущности не отслеживались - нужно добавить к оператору LINQ вызов AsNoTracking(). 

Пример:

```csharp
var one = context.Samples.AsNoTracking().FirstOrDefault(x => x.Id == 1);
```

Чтобы при этом не создавались добавочные копии сущностей, нужно вызвать метод AsNoTrackingWithIdentityResolution().

Пример:

```csharp
var one = context.Samples.AsNoTrackingWithIdentityResolution().FirstOrDefault(x => x.Id == 1);
```











