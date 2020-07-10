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
        Database.SetInitializer<AutoLotEntities>(new MyDataInitializer());
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


## Перехватчики ObjectMaterialized и SavingChanges

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





