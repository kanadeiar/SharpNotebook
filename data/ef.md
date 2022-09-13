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

### Свойство типа DBSet<T>

Для каждой таблицы в объектной модели нужно предусмотреть свойство типа DbSet<T>. Для перевода свойств в режим ленивой загрузки эти свойства должны быть виртуальными.
    
Некоторые члены DBSet<T>:

Член                     | Описание
-------------------------|-------------------------
Add() AddRange()         | Позволяют вставлять в коллекцию новый объект (или пачку). Они получают статус EntityState.Added и будут добавлены в базу от SaveChanges() на DbContext
Attach()                 | Ассоциирует объект с DbContext. Применяется в автономных приложениях ASP.NET|MVC
Create() Create<T>()     | Новый экземпляр указанного сущностного типа
Find() FindAsync()       | Находит строку данных по первичному ключу и возвращает объект, представляющий эту строку.
Remove() RemoveRange()   | Помечает объект - или группу - для последующего удаления
SqlQuery()               | Создает низкоуровневый запрос SQL, возвращяющий сущности в этом наборе

Все EF помещают свои вызовы SaveChanges()/SaveChangesAsync() внутрь транзакции. Внутрь неявной транзакции помещается и ExecuteSqlCommand().

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


















