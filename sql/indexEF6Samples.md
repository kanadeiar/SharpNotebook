# EF 6 Примеры

## Библиотека классов, работающая с инфраструктурой EF.

1. Добавить NuGet пакет EF 6. 

2. Создать классы - сущности - копии сгенерированные мастером.

3. Добавить строку подключения в App.config.

### Инициализация базы данных

Мощным средством EF является возможность обеспечить равенстро базы данных и модели, инициализировать базу данных начальными данными. Это позволяет восстановить базу данных в известном состоянии перед каждым запуском кода.

Для инициализации необходимо создать класс производный от DropCreateDatabaseIfModelChanges<TContext> или DropCreateDatabaseAlways<TContext>.
    
Пример базовой версии библиотеки, удаляющей и воссоздающей базу данных каждый раз, когда выполняется инициализатор:
```csharp
using System.Data.Entity;
public partial class AutoLotEntities : DbContext
{ //модель сущностей
    public AutoLotEntities()
        : base("name=AutoLotConnection")
    {
        Database.SetInitializer<AutoLotEntities>(new MyDataInitializer()); //удаление и инит базы данных
    }
    public virtual DbSet<CreditRisk> CreditRisks { get; set; }
    public virtual DbSet<Customer> Customers { get; set; }
    public virtual DbSet<Inventory> Inventories { get; set; }
    public virtual DbSet<Order> Orders { get; set; }
    protected override void OnModelCreating(DbModelBuilder modelBuilder)
    {
    }
}
public class MyDataInitializer : DropCreateDatabaseAlways<AutoLotEntities>
{ 
    protected override void Seed(AutoLotEntities context)
    { //начальные данные 
        var customers = new List<Customer>
        {
            new Customer {FirstName = "Dave", LastName = "Brenner"},
...
        };
        customers.ForEach(x => context.Customers.AddOrUpdate(c => new {c.FirstName, c.LastName}, x));
        var cars = new List<Inventory>
        {
            new Inventory {Make = "VW", Color = "Black", Name = "Zippy"},
...
        };
        context.Inventories.AddOrUpdate(x => new {x.Make, x.Color}, cars.ToArray());
        var orders = new List<Order>
        {
            new Order { Inventory = cars[0], Customer = customers[0]},
...
        };
        orders.ForEach(x => context.Orders.AddOrUpdate(c => new {c.CarId, c.CustId}, x));
        context.CreditRisks.AddOrUpdate(x => new { x.FirstName, x.LastName },
            new CreditRisk
            { CustId = customers[4].CustId, FirstName = customers[4].FirstName, LastName = customers[4].LastName, });
    }
}
[Table("Inventory")]
public partial class Inventory
{
    [System.Diagnostics.CodeAnalysis.SuppressMessage("Microsoft.Usage", "CA2214:DoNotCallOverridableMethodsInConstructors")]
    public Inventory()
    { }

    [Key]
    public int CarId { get; set; }

    [StringLength(50)]
    public string Make { get; set; }

    [StringLength(50)]
    public string Color { get; set; }

    [StringLength(50)]
    public string Name { get; set; }

    [System.Diagnostics.CodeAnalysis.SuppressMessage("Microsoft.Usage", "CA2227:CollectionPropertiesShouldBeReadOnly")]
    public virtual ICollection<Order> Orders { get; set; } = new HashSet<Order>();
}
public partial class Customer
{
    [System.Diagnostics.CodeAnalysis.SuppressMessage("Microsoft.Usage", "CA2214:DoNotCallOverridableMethodsInConstructors")]
    public Customer()
    { }

    [Key]
    public int CustId { get; set; }

    [StringLength(50)]
    public string FirstName { get; set; }

    [StringLength(50)]
    public string LastName { get; set; }

    [System.Diagnostics.CodeAnalysis.SuppressMessage("Microsoft.Usage", "CA2227:CollectionPropertiesShouldBeReadOnly")]
    public virtual ICollection<Order> Orders { get; set; } = new HashSet<Order>();
}
public partial class Order
{
    [Key]
    public int OrderId { get; set; }

    public int CustId { get; set; }

    public int CarId { get; set; }

    public virtual Customer Customer { get; set; }

    public virtual Inventory Inventory { get; set; }
}
public partial class CreditRisk
{
    [Key]
    public int CustId { get; set; }

    [StringLength(50)]
    public string FirstName { get; set; }

    [StringLength(50)]
    public string LastName { get; set; }
}
//использование:
static void Main(string[] args)
{
    Database.SetInitializer(new MyDataInitializer());
    Console.WriteLine("Список:");
    using (var context = new AutoLotEntities())
    {
        foreach (var el in context.Inventories)
            Console.WriteLine(el);
    }
    Console.WriteLine("Нажмите любую кнопку ...");
    Console.ReadKey();
}
```

### Миграции EF

Если модель данных изменяется, то необходимо поддерживать базу данных в синхронизированном состоянии. Для этого существуют миграции EF. 

При создании и применении контекста для выполнения операций в базе данных контекст проверяет существование таблицы __MigrationHistory, и когда таблица есть, он сравнивает хеш текущей модели EF и хеш, сохраненный в последней записи таблицы. Если отличаются - генерируется исключение.

### Включение миграций

Миграции должны быть включечны, так как при отсутствии таблицы миграций генерятся куча затратных исключений. 

Включить "Вид->Другие окна->Консоль диспетчера пакетов".

Установить стандартный проект - библиотека - команда (EF 6.2): 
```csharp
enable-migrations
```
В методе Seek() можно добавлять изначальные данные.

Создание начальной миграции (создается в папке Migrations дополнительный файл со спец именем, внутри него - Up() для применения изменений в базе данных, Down() - для отката изменений):
```csharp
add-migration Initial
```

Обновление базы данных в соответствии с моделью (создание базы данных, таблиц и таблицы __MigrationHistory):
```csharp
update-database
```

После внесения каких-либо измененений в модель базы данных, можно создать дополнительную миграцию конечную.
```csharp
add-migration Final
```

Возможно придется переопределить порядок выполнения команд миграции.

### Паттерн "Хранилище"

Можно использовать в библиотеке паттерн проектирования "хранилище" для доступа к данным. Суть паттерна - служить посредником между предметной областью и уровнями отображения данных.


Интерфейс, открывающий доступ к большинству стандартных методов библиотеки доступа к данным:
```csharp
/// <summary> Интерфейс хранилища </summary>
public interface IRepo<T>
{
    int Add(T entity);
    int AddRange(IList<T> entities);
    int Save(T entity);
    int Delete(int id, byte[] timeStamp);
    int Delete(T entity);
    T GetOne(int? id);
    List<T> GetAll();
    List<T> ExecuteQuery(string sql);
    List<T> ExecuteQuery(string sql, object[] sqlParametersObjects);
}
```

Базовый класс реализующий этот интерйейс и предоставляющий основную функциональность для хранилищ, специфическим к типам:
```csharp
/// <summary> Базовый класс хранилища </summary>
public class BaseRepo<T> : IDisposable, IRepo<T> where T:EntityBase, new()
{
    private readonly AutoLotEntities _entities;
    private readonly DbSet<T> _table;
    public BaseRepo()
    {
        _entities = new AutoLotEntities();
        _table = _entities.Set<T>();
    }
    protected AutoLotEntities Context => _entities;
    public void Dispose() => _entities?.Dispose();
    internal int SaveChanges()
    {
        try
        {
            return _entities.SaveChanges();
        }
        catch (DbUpdateConcurrencyException ex)
        { //ошибка параллелизма
            throw;
        }
        catch (DbUpdateException ex)
        { //обновление терпит неудачу
            throw;
        }
        catch (CommitFailedException ex)
        { //отказы транзакции
            throw;
        }
        catch (Exception ex)
        { //другое исключение
            throw;
        }
    }
    /// <summary> Извлечение одной записи </summary>
    public T GetOne(int? id) => _table.Find(id);
    /// <summary> Извлечение всех записей </summary>
    public List<T> GetAll() => _table.ToList();
    /// <summary> Извлечение с помощью кода SQL ОПАСНО! </summary>
    public List<T> ExecuteQuery(string sql) => _table.SqlQuery(sql).ToList();

    /// <summary> Извлечение с помощью кода SQL ОПАСНО! </summary>
    public List<T> ExecuteQuery(string sql, object[] sqlParametersObjects) 
        => _table.SqlQuery(sql, sqlParametersObjects).ToList();
    /// <summary> Добавление одной записи </summary>
    public int Add(T entity)
    {
        _table.Add(entity);
        return SaveChanges();
    }
    /// <summary> Добавление кипы записей </summary>
    public int AddRange(IList<T> entities)
    {
        _table.AddRange(entities);
        return SaveChanges();
    }
    /// <summary> Обновление записей </summary>
    public int Save(T entity)
    {
        _entities.Entry(entity).State = EntityState.Modified;
        return SaveChanges();
    }
    /// <summary> Удаление записей </summary>
    public int Delete(int id, byte[] timeStamp)
    {
        _entities.Entry(new T() {Id = id, Timestamp = timeStamp}).State = EntityState.Deleted;
        return SaveChanges();
    }
    /// <summary> Удаление записей </summary>
    public int Delete(T entity)
    {
        _entities.Entry(entity).State = EntityState.Deleted;
        return SaveChanges();
    }
}
```

Возможно в хранилищах, специфичных для сущности предусмотреть дополнительную функциональность. Пример:
```csharp
class InventoryRepo : BaseRepo<Inventory>
{
    /// <summary> Извлечение всех записей </summary>
    public new List<Inventory> GetAll() => Context.Inventories.OrderBy(x => x.Name).ToList();
}
```

### Модель хранилища

В хранилище используется следующая модель:
```csharp
namespace AutoLotDAL.EF
{
    using System.Data.Entity;

    public partial class AutoLotEntities : DbContext
    {
        public AutoLotEntities()
            : base("name=AutoLotConnection")
        {
        }
        public virtual DbSet<CreditRisk> CreditRisks { get; set; }
        public virtual DbSet<Customer> Customers { get; set; }
        public virtual DbSet<Inventory> Inventories { get; set; }
        public virtual DbSet<Order> Orders { get; set; }
        protected override void OnModelCreating(DbModelBuilder modelBuilder)
        {
        }
...
namespace AutoLotDAL.EF
{
    public class MyDataInitializer : DropCreateDatabaseAlways<AutoLotEntities>
    {
        protected override void Seed(AutoLotEntities context)
        {
            var customers = new List<Customer>
            {
                new Customer {FirstName = "Dave", LastName = "Brenner"},
...
            };
            customers.ForEach(x => context.Customers.AddOrUpdate(c => new {c.FirstName, c.LastName}, x));
            var cars = new List<Inventory>
            {
                new Inventory {Make = "VW", Color = "Black", Name = "Zippy"},
...
            };
            context.Inventories.AddOrUpdate(x => new {x.Make, x.Color}, cars.ToArray());
            var orders = new List<Order>
            {
                new Order { Inventory = cars[0], Customer = customers[0]},
...
            };
            orders.ForEach(x => context.Orders.AddOrUpdate(c => new {c.CarId, c.CustId}, x));
            context.CreditRisks.AddOrUpdate(x => new { x.FirstName, x.LastName },
                new CreditRisk
                { Id = customers[4].Id, FirstName = customers[4].FirstName, LastName = customers[4].LastName, });
        }
    }
}
namespace AutoLotDAL.Models.Base
{
    public class EntityBase
    {
        [Key] //добавление в каждую сущность
        public int Id { get; set; }
        [Timestamp] //добавление в каждую сущность
        public byte[] Timestamp { get; set; }
...
namespace AutoLotDAL.EF
{
    [Table("Inventory")]
    public partial class Inventory : EntityBase
    {
        [System.Diagnostics.CodeAnalysis.SuppressMessage("Microsoft.Usage", "CA2214:DoNotCallOverridableMethodsInConstructors")]
        public Inventory()
        { }
        [StringLength(50)]
        public string Make { get; set; }
        [StringLength(50)]
        public string Color { get; set; }
        [StringLength(50)]
        public string Name { get; set; }
        [System.Diagnostics.CodeAnalysis.SuppressMessage("Microsoft.Usage", "CA2227:CollectionPropertiesShouldBeReadOnly")]
        public virtual ICollection<Order> Orders { get; set; } = new HashSet<Order>();
...
    public partial class Inventory
    {
        public override string ToString()
        {
            return $"{Name ?? "<без имени>"} цвета {Color} марки {Make} c Id {Id}";
        }
        [NotMapped]
        public string MakeColor => $"{Make} + {Color}";
...
namespace AutoLotDAL.EF
{

    public partial class Customer : EntityBase
    {
        [System.Diagnostics.CodeAnalysis.SuppressMessage("Microsoft.Usage", "CA2214:DoNotCallOverridableMethodsInConstructors")]
        public Customer()
        { }
        [StringLength(50)]
        public string FirstName { get; set; }
        [StringLength(50)]
        public string LastName { get; set; }
        [System.Diagnostics.CodeAnalysis.SuppressMessage("Microsoft.Usage", "CA2227:CollectionPropertiesShouldBeReadOnly")]
        public virtual ICollection<Order> Orders { get; set; } = new HashSet<Order>();
...
    public partial class Customer
    {
        [NotMapped]
        public string FullName => FirstName + " " + LastName;
    }
namespace AutoLotDAL.EF
{
    using System.ComponentModel.DataAnnotations.Schema;

    public partial class Order : EntityBase
    {
        public int CustId { get; set; }
        public int CarId { get; set; }
        [ForeignKey(nameof(CustId))]
        public virtual Customer Customer { get; set; }
        [ForeignKey(nameof(CarId))]
        public virtual Inventory Inventory { get; set; }
...
namespace AutoLotDAL.EF
{
    using System.ComponentModel.DataAnnotations;

    public partial class CreditRisk : EntityBase
    {
        [StringLength(50)]
        [Index("IDX_CreditRisk_Name", IsUnique = true, Order = 2)]
        public string FirstName { get; set; }
        [StringLength(50)]
        [Index("IDX_CreditRisk_Name", IsUnique = true, Order = 1)]
        public string LastName { get; set; }
...
```


### Использование хранилища

>В клиенское приложение установить пакет EF (сборки EntityFramework.dll, EntityFramework.SqlServer.dll).

Пример исползьзования в клиенском приложении:
```csharp
using AutoLotDAL.Repos;
...
static void Main(string[] args)
{
    using (var repo = new InventoryRepo())
    {
        foreach (var el in repo.GetAll())
        {
            Console.WriteLine(el);
        }
    }
/// <summary> Добавление </summary>
private static void AddNewRecord(Inventory inventory)
{
    using (var repo = new InventoryRepo())
    {
        repo.Add(inventory);
    }
}
/// <summary> Изменение </summary>
private static void UpdateRecord(int carId)
{
    using (var repo = new InventoryRepo())
    {
        var updateInv = repo.GetOne(carId);
        if (updateInv == null) return;
        updateInv.Color = "Blue";
        repo.Save(updateInv);
    }
}
/// <summary> Удаление </summary>
private static void RemoveInv(Inventory deleteInventory)
{
    using (var repo = new InventoryRepo())
    {
        repo.Delete(deleteInventory);
    }
}
/// <summary> Удаление по свойствам ключа </summary>
private static void RemoveInv(int carId, byte[] timeStamp)
{
    using (var repo = new InventoryRepo())
    {
        repo.Delete(carId, timeStamp);
    }
}
```

### Параллелизм в хранилище

Когда два пользователя обновляют одну и ту же запись, то остаются изменения последнего из них. EF дают удобный механизм проверки наличия конфликтов параллелизма.

Аннотация данных Timestamp командует SQL Server создать столбец типа RowVersion. Значение в этом столбце обновляется при SQL Server при каждом добавлении или обновлении записи. Это изменяет и запросы EF к базе данных. Теперь если несколько пользователей изменяет одну и туже запись, то только у первого пользователя значение столбца Timestamp совпадет и запись будет изменена. У остальных - произойдет исключение DbUpdateCuncurrencyException. Это класс - исключение открывает доступ к коллекции Entries с сущностями, с которыми произошел облом.

Пример обработки исключения параллелизма в приведенном примере:
```csharp
/// <summary> Тест параллелизма </summary>
private static void TestConcurrency()
{
    var repo1 = new InventoryRepo();
    var repo2 = new InventoryRepo();
    var car1 = repo1.GetOne(1);
    var car2 = repo2.GetOne(1);
    car1.Name = "NewName";
    repo1.Save(car1);
    car2.Name = "OtherName";
    try
    {
        repo2.Save(car2);
    }
    catch (DbUpdateConcurrencyException ex)
    {
        var entry = ex.Entries.Single();
        var current = entry.CurrentValues;
        var original = entry.OriginalValues;
        var baseVal = entry.GetDatabaseValues();
        Console.WriteLine("Exception");
        Console.WriteLine($"Current: {current[nameof(Inventory.Name)]}");
        Console.WriteLine($"Orig: {original[nameof(Inventory.Name)]}");
        Console.WriteLine($"base: {baseVal[nameof(Inventory.Name)]}");
    }
}
```

### Перехват

Перехват в EF - выполнение различного кода на разных стадиях работы EF. Используется интерфейс IDBCommandInterceptor.

Пример добавления перехватичка в приведенном примере:
```csharp
namespace AutoLotDAL.Interception
{
    /// <summary> Перехватчик </summary>
    class ConsoleWriterInterceptor : IDbCommandInterceptor
    {
        public void NonQueryExecuting(DbCommand command, DbCommandInterceptionContext<int> interceptionContext)
        {
            WriteInfo(interceptionContext.IsAsync, command.CommandText);
        }
... //то-же самое
        private void WriteInfo(bool isAsync, string sql)
        {
            Console.WriteLine($"Асинхрон: {isAsync}, запрос: \n{sql}");
        }
...
//регистрация перехватчика:
namespace AutoLotDAL.EF
{
    public partial class AutoLotEntities : DbContext
    {
        public AutoLotEntities()
            : base("name=AutoLotConnection")
        {
            DbInterception.Add(new ConsoleWriterInterceptor());
        }
...
```

В EF есть встроенный перехватчик, выполняющий простую регистрацию в текстовом журнале.

Пример добавления такого перехватчика в приведенном примере:
```csharp
namespace AutoLotDAL.EF
{
    using System.Data.Entity;
    public partial class AutoLotEntities : DbContext
    {
        static readonly DatabaseLogger DatabaseLogger= new DatabaseLogger("sqllog.txt", true);
        public AutoLotEntities()
            : base("name=AutoLotConnection")
        {
            //DbInterception.Add(new ConsoleWriterInterceptor());
            DatabaseLogger.StartLogging();
            DbInterception.Add(DatabaseLogger);
        }
        protected override void Dispose(bool disposing)
        {
            DbInterception.Remove(DatabaseLogger);
            DatabaseLogger.StopLogging();
            base.Dispose(disposing);
        }
```

Событие 



### Перехватчики ObjectMaterialized и SavingChanges

Событие ObjectMaterialized - реконструирование объекта из хранилища данных, перед возвращением экземпляра вызывающему коду. 

Событие SavingChanges - распорстранение данных объекта в хранилище, сразу после SaveChanges().

Пример перехвата событий и отклонений изменений свойств базы данных:
```csharp
namespace AutoLotDAL.EF
{
    public partial class AutoLotEntities : DbContext
    {
        static readonly DatabaseLogger DatabaseLogger = new DatabaseLogger("sqllog.txt", true);
        public AutoLotEntities()
            : base("name=AutoLotConnection")
        {
            var context = (this as IObjectContextAdapter).ObjectContext;
            context.ObjectMaterialized += OnObjectMaterialized;
            context.SavingChanges += OnSavingChanges;
        }
        private void OnSavingChanges(object sender, EventArgs eventArgs)
        { //текущие, исходные значения, можно отменять/модифицировать операции сохранения любым способом
            var context = sender as ObjectContext;
            if (context == null) return;
            foreach (var item in context.ObjectStateManager.GetObjectStateEntries(EntityState.Modified | EntityState.Added))
            { //добавленные или измененнные
                if ((item.Entity as Inventory) != null)
                {
                    var entity = (Inventory)item.Entity;
                    if (entity.Name == "NewName")
                    { //отклоняет все изменения названий в NewName
                        item.RejectPropertyChanges(nameof(entity.Name));
                    }
                }
            }
        }
        private void OnObjectMaterialized(object sender, System.Data.Entity.Core.Objects.ObjectMaterializedEventArgs e)
        { //полезно при работе с WPF
        }
...
```


### Отделение модели от уровня доступа к данным

В некоторых проектах оказывается чрезвычайно полезным отделить модель от кода EF. Делается это путем перемещения моделей в отдельную библиотеку, которая будет изменяться отдельно от основного кода EF.

Пример библиотеки, разделенной на две сборки - модель и код EF (без хранилища и миграций):
```csharp
namespace SampleDAL.EF
{
    public partial class SampleEntities : DbContext
    {
        public SampleEntities() : base("name=SampleConnection")
        {
        }
        public virtual DbSet<Person> Persons { get; set; }
        public virtual DbSet<Department> Departments { get; set; }
        protected override void OnModelCreating(DbModelBuilder modelBuilder)
        {
        }
...
namespace SampleDAL.EF
{
    public class SampleDataInitializer : DropCreateDatabaseAlways<SampleEntities>
    {
        protected override void Seed(SampleEntities context)
        {
            var departments = new List<Department>
            {
                new Department {Name = "Отдел быта"},
...
            };
            departments.ForEach(x => context.Departments.AddOrUpdate(d => new {d.Name}, x));
            var persons = new List<Person>
            {
                new Person {Fam = "Петров", Name = "Петя", Age = 18, Department = departments[0]},
...
            };
            persons.ForEach(x => context.Persons.AddOrUpdate(p => new {p.Fam, p.Name, p.Age, p.DepId}, x));
        }
...
namespace SampleDAL.Models.Base
{
    public class BaseEntity
    {
        [Key]
        public int Id { get; set; }
        [Timestamp]
        public byte[] Timestamp { get; set; }
    }
...
namespace SampleDAL.Models
{
    [Table("Department")]
    public partial class Department : BaseEntity
    {
        public Department() {}
        [StringLength(50)]
        public string Name { get; set; }
        public virtual ICollection<Person> Persons { get; set; } = new HashSet<Person>();
    }
...
namespace SampleDAL.Models
{
    public partial class Department
    {
        public override string ToString()
        {
            return $"Название отдела: {Name ?? "без имени"}";
        }
    }
...
namespace SampleDAL.Models
{
    [Table("Person")]
    public partial class Person : BaseEntity
    {
        public Person() {}

        [StringLength(50)]
        public string Fam { get; set; }
        [StringLength(50)]
        public string Name { get; set; }
        public int Age { get; set; }
        public int DepId { get; set; }
        [ForeignKey(nameof(DepId))]
        public virtual Department Department { get; set; }
    }
...
namespace SampleDAL.Models
{
    public partial class Person
    {
        public override string ToString()
        {
            return $"id: {Id} {Fam ?? "без фамилии"} {Name ?? "без имени"} {Age}";
        }
        [NotMapped]
        public string FullName => Fam + " " + Name;
    }
...
namespace ConsoleApp2
{
    class Program
    {
        static void Main(string[] args)
        {
            //Database.SetInitializer(new SampleDataInitializer()); //удаление, создание и заполнение базы данных
            using (var context = new SampleEntities())
            {
                Console.WriteLine("Persons:");
                foreach (var per in context.Persons)
                    Console.WriteLine(per + " - " + per.Department.Name);
                Console.WriteLine("Departments:");
                foreach (var dep in context.Departments)
                {
                    Console.WriteLine(dep + ":");
                    foreach (var per in dep.Persons)
                        Console.WriteLine(per);
                }
            }
            Console.WriteLine("Нажмите кнопку ...");
            Console.ReadKey();
        }
...
// App.config самого приложения
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <configSections>
...
  </entityFramework>
  <connectionStrings>
    <add name="SampleConnection" connectionString="data source=(localdb)\MSSQLLocalDB;initial catalog=sample;integrated security=True;MultipleActiveResultSets=True;App=EntityFramework" providerName="System.Data.SqlClient" />
  </connectionStrings>
</configuration>
```


