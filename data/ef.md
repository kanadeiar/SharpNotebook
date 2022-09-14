# Entity Framework

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

Существует возможность возвращать производный DbContext в исходное состояние. Метод ChangeTracker.Clear() удаляет все объекты из коллекций DbSet<T>, устанавливая их состояние в состояние Detached.

## Сущности






## Типы запросов

Типы запросов — это коллекции DbSet<T>, которые применяются для изображения представлений, оператора SQL или таблиц без первичного ключа. Типы запросов добавляются к производному классу DbContext с применением свойств DbSet<T> и конфигурируются как не имеющие ключей.

Требуемый класс SampleViewModel (который вы создадите при построении полной библиотеки доступа к данным AutoLot)конфигурируется с атрибутом [Keyless].

```csharp
[Keyless]
public class SampleViewModel
{
}
```

Остальные действия по конфигурированию делаются в Fluent API.

## Fluent API
















