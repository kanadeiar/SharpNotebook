# Введение EF 6

В версии .NET 3.5 SP1 был добавлен компонент API-интерфейса ADO.NET под названием Entity Framework (EF). Цель компонента этого - дать возможность взаимодействия с данными из реляционных баз данных в виде объектной модели, отображаемой прямо на бизнес-объекты в приложении. Оперировать коллекцией строго типизированных объектов, называемых сущностями (entity). 

Инфраструктура EF сокращает разрыв между целями и оптимизацией базы данных и целями и оптимизацией объектно-ориентированного программирования. С помощью EF можно работать с базами данных, не сталкиваясь со строками кода SQL. Выполняемая среда EF самостоятельно генерирует подходящие операторы SQL.

## Составные части EF

Строго типизированные классы называемые сущностями - модули физической базы данных, отображаемые на предметную область. Формально - модель сущностных данных (Entity Data Model - EDM). Сущности не обязаны напрямую отображаться на схуму базы данных, обычто так не бывает. Сущностные классы делаются для удовлетворения нужно приложения, затем отображаются на имеющеюся базу данных.

За кулисами работы с EF инфраструктура создает и открывает подключение к базе данных, генерируется и выполняется подходящий оператор SQL, подключение освобождается и закрывается. Обрабатывается без участия человека.

Инфраструктура и ядро EF при работе использует ADO.NET. Требуется поставщик данных, поддерживающий эту инфраструктуру. Например System.Data.Entity для Microsoft SQL Server. 

Класс DbContext используется для запрашивания базы данных и объединения изменений, записивыемых обратно в базу данных в виде одинойной еденицы работы. Класс предоставляет дочерним службы для сохранения изменений, настройку строки подключения, удаление объектов, вызов хранимых процедур и прочее.

Некоторые члены DbContext:

Член                     | Описание
-------------------------|-------------------------
DbContext()              | Конструктор, применяемый в производном контекстном классе. В строковом параметре - либо имя базы данных, либо строка подключения в файле *.config.
Entry() Entry<TEntity>() | Извлекает объект System.Data.Entity.Infrastructure.DbEntityEntry, предоставляющий доступ к информации и возможность выполнения действий над сущностью.
GetValidationErrors()    | Проверяет достоверность обрабатываемых сущностей и возвращает коллекцию объектов System.Data.Entity.Validation.DbEntityValidationResult
SaveChanges() SaveChangesAsync() | Сохраняет в базе данных все изменения контекста, вертает кол-во затронутых записей.
Configuration()          | Доступ к свойствам конфигурации контекста
Database                 | Дает механизм для создания/удаления/проверки существования лежащей в основе базы данных, для выполнения хранимых процедур и низкоуровневых опереторов SQL над базой данных и для доступа работы с транзакциями.

Событие                  | Описание
-------------------------|-------------------------
ObjectMaterialized       | Инициируется при создании нового сущностного объекта из хранилища данных запросом выборки или загрузкой
SavingChanges            | Происходит при сохранении изменений в базе данных, но перед моментом окончательной записи изменений в базу данных

### Парадигма Code First

Класс DbContext предоставляет основную функциональность во время работы со средством CodeFirst инфраструктуры EF. Это не означает невозможность примеения EF с существующей базой данных. Можно как использовать Code First с существующей базой данных или создать новую базу данных на основе сущностей с помощью миграций EF.

### Производный контекстный класс

Пример производного класса от DbContext специфичной предметной области, с переданной строкой подключения:
```csharp
public class SampleEntities : DbContext
{
    public SampleEntities() : base("name = SampleConnection")
    { }
    protected override void Dispose(bool disposing)
    { }
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
    
### Транзакции

Все EF помещают свои вызовы SaveChanges()/SaveChangesAsync() внутрь транзакции. Начиная с версии EF 6, внутрь неявной транзакции помещается и ExecuteSqlCommand().

### Состояние сущности

Любые изменения в данных будут отслеживатся и записываться при вызове SaveShanges().

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

### Аннотации данных

Аннотации данных C# применяются для придания формы сущностям. Доступно множество аннотаций данных для уточнения модели и добавления проверок достоверности.

Некоторые аннотации данных, поддерживаемых EF:

Аннотация данных         | Описание
-------------------------|-------------------------
Key                      | Первичный ключ для модели. Необящательно, если свойство ключа именовано как Id, или комбинация имени с Id (MyId). При составном ключе нужно добавить атрибут Column с параметром нужным (Column[My=1] Column[My=2]). Являются неявно [Required]
Required                 | Объявляет свойство как не допускающие null
ForeignKey               | Свойство, применяемое как внешний ключ для навигационного свойства
StringLength             | Минимальная и максимальная длина строкового свойства
NotMapped                | Свойство, которое не отображается на поле базы данных
ConcurrencyCheck         | Поле, используемое при проверке парралелизма, когда сервер выполняет обновления, вставку или удаление данных.
TimeStamp                | Тип как версию строки или отметку времени
Table Column             | Именование классов и полей моделей не так, как они объявлены в базе данных. Атрибут Table - спецификацию схемы.
DatabaseGenerated        | Указание, что поле сгенерировано базой данных - Computed, Identity, None.
Index                    | Указание, что столбец должен иметь индекс, возможно указать кластеризацию, уникальность, имя, порядок

## Создание модели сущностных классов

? Запуск VS под админиcтратором

? Добавить в проект ссылку на System.Data.Entity.

? Установить пакет NuGet EntityFramework 6.

В папку добавить новый элемент - "Данные" -> "Модель ADO.NET EDM" - название "*Entities".

Выбрать "CodeFirst из базы данных".

Выбрать подключение к базе данных - сохранить - установить название "*Connection".

Отметить все интересующие таблицы, кроме "sysdiagrams", отметить галку "Формировать имена объектов во множественном или единственном числе".

Исследовать полученные классы и модифицировать их под задачу. В файле "App.config" - новый элемент "EntityFramework" и строка подключения к базе данных.

Добавление новых возможностей в сгенерированные классы доступно только путем создания дополнительных файлов с таким названием класса и "partial", т.к. все сгенерированные классы "partial" и когда классы будут генерироватся повторно, не возникнет риска потери изменений. Например файл "CarPartial.cs":
```csharp
public partial class Inventory
{
    public override string ToString()
    {
        return $"{this.Name ?? "<без имени>"} цвета {this.Color} марки {this.Make} c Id {this.CarId}";
    }
}
```

## Использование этой модели

### Выборка записей

Способов получения записей из базы данных с использованием EF существует множество. 

Достоинство инфраструктуры EF в том, что запросы выборки не выполняются до тех пор, пока не начнется проход по результирующей коллекции или при вызове на нем методов ToList() или ToArray().

Простейший способ - проход по коллекции DBSet<Inventory>:
```csharp
private static void PrintAllInventory()
{
    using (var context = new AutoLotEntities())
    {
        foreach (var el in context.Inventories)
            Console.WriteLine(el);
    }
}
```
    
Посложнее способ - извлечение сущностей из базы через встраиваемый запрос SQL (данный способ требует дополнительного сопровождения при изменениях базы данных):
```csharp
private static void SqlPrintAllInventory()
{
    using (var context = new AutoLotEntities())
    {
        foreach (var el in context.Inventories.SqlQuery("SELECT CarId, Make, Color, Name FROM Inventory WHERE Make=@p0", "BMW"))
            Console.WriteLine(el);
    }
}
```

Гибкий способ - использование метда SqlQuery() на свойстве Database для заполнения сущностей EF и объектов, не являющихся частью модели.
```csharp
public class ShortCar
{
    public int CarId { get; set; }
    public string Make { get; set; }
    public override string ToString() => $"{this.Make} с Ид {this.CarId}";
}
private static void ShortPrintAllInventory()
{
    using (var context = new AutoLotEntities())
    {
        foreach (var el in context.Database.SqlQuery(typeof(ShortCar),"SELECT CarId, Make FROM Inventory"))
            Console.WriteLine(el);
    }
}
```

Мощный способ - объединение EF и LINQ. В этом способе оператор LINQ транслируется в один запрос SQL. Примеры:
```csharp
private static void LINQPrintAllInventory()
{
    using (var context = new AutoLotEntities())
    {
        foreach (var el in context.Inventories.Where(i => i.Make == "BMW"))
            Console.WriteLine(el);
        Console.WriteLine("новые данные:");
        var colorMakes = context.Inventories.Select(i => new {i.Color, i.Make});
        foreach (var el in colorMakes)
            Console.WriteLine(el);
        var colorBlack = context.Inventories.Where(i => i.Color == "Black");
        foreach (var el in colorBlack)
            Console.WriteLine(el);
    }
}
```

Еще способ - нахождение сущности Find(). Этот метод сначало ищет сущность в DbChangesTracker, и только в случае ее отсутствия, запрашивает из базы данных. Ищет только по первичному ключу (простому или сложному).

Пример:
```csharp
using (var context = new AutoLotEntities())
{
    Console.WriteLine(context.Inventories.Find(9));
}
```

### Навигационные свойства

Навигационные свойства позволяют находить связанные данные в других сущностях без необходимости в написании запросов JOIN. EF учитывает отношения между таблицами SQL Server как внешние ключи. 

В EF каждый класс в модели содержит виртуальные свойства, которые соединяют вместе классы. У одного класса это свойство может иметь как ноль, так и множество записей другой сущности. А у другого класса - свойство имеющее отношение "один-к-одному". 
```csharp
//Inventory
public virtual ICollection<Order> Orders { get; set; }
//Order
public virtual Inventory Inventory { get; set; }
```
По соглашению, когда EF находит свойство по имени <Класс>Id, оно будет применяться в качестве внешнего ключа для навигационного свойства <Класс>.

Загрузка данных из БД в сущностные классы может производится тремя разными способами:

Ленивая загрузка:

Инфраструктура EF загружаает затребованный непосредственный объект, но заменяет навигационные свойства посредниками. Когда запрашивается свойство на навигационном свойстве virtual, EF создает новое обращение к базе данных, выполняет его и заполняет детали объекта.

Пример:
```csharp
using (var context = new AutoLotEntities())
{
    var inv = context.Inventories.First();
    foreach (var el in inv.Orders)
    {
        Console.WriteLine($"{inv.Name} - id: {el.OrderId}");
    }
}
```

Энергичная загрузка:

Когда нужно загрузить связанные записи, множество запросов к базе данных будет неэффективным, здесь нужно написать один запрос, утилизирующий соединения SQL. Метод Include() из LINQ информирует EF о надобности создания оператора SQL, соединяющий таблицы вместе (JOIN). В результате все объекты и связанные с ним зависимые объекты извлекаются за одно обращение к базе данных.

Пример:
```csharp
using (var context = new AutoLotEntities())
{
    foreach (var inv in context.Inventories.Include(c => c.Orders))
    {
        foreach (var el in inv.Orders)
        {
            Console.WriteLine($"{inv.Name} - id: {el.OrderId}");
        }
    }
}
```

Явная загрузка:

Это способ явной загрузки коллекции или класса в конец навигационного свойства. Методы Collection() для коллекций или Property() для одиночных объектов контекста, и метод Load().

Пример:
```csharp
using (var context = new AutoLotEntities())
{
    context.Configuration.LazyLoadingEnabled = false;
    foreach (var inv in context.Inventories)
    {
        context.Entry(inv).Collection(x => x.Orders).Load();
        foreach (var el in inv.Orders)
        {
            Console.WriteLine($"{inv.Name} - id: {el.OrderId}");
        }
    }
    Console.WriteLine("Orders:");
    foreach (var el in context.Orders)
    {
        context.Entry(el).Reference(x => x.Inventory).Load();
        Console.WriteLine($"id: {el.OrderId} {el.Inventory.Make} {el.Inventory.Color}");
    }
}
```


### Добавление записей

Добавление новой записи сводится к добавлению записей в DbSet<T> и вызову SaveChanges(). 
    
Пример:
```csharp
private static int AddNewRecord()
{
    using (var context = new AutoLotEntities())
    {
        try
        {
            var car = new Inventory() { Make = "Yugo", Color = "White", Name = "Green" };
            context.Inventories.Add(car);
            context.SaveChanges();
            return car.CarId;
        }
        catch (Exception ex)
        {
            Console.WriteLine(ex.InnerException?.Message);
            return 0;
        }
    }
}
int carId = AddNewRecord();
```

Пример добавления можножества записей:
```csharp
private static void AddNewRecords(IEnumerable<Inventory> cars)
{
    using (var context = new AutoLotEntities())
    {
        context.Inventories.AddRange(cars);
        context.SaveChanges();
    }
}
Inventory[] inventories =
{
    new Inventory{Make = "Test1", Color = "Bl1", Name = "HotName1"},
    new Inventory{Make = "Test2", Color = "Bl2", Name = "HotName2"},
};
AddNewRecords(inventories);
```

### Обновление записей

Для обновления нужно определить местоположение объекта, установить новые значения для свойств сущности и сохранить изменения.

Пример:
```csharp
private static void UpdateRecord(int carId)
{
    using (var context = new AutoLotEntities())
    {
        var upd = context.Inventories.Find(carId);
        if (upd != null)
        {
            upd.Name = "Blue";
            context.SaveChanges();
        }
    }
}
```

### Удаление записей

Удаление может быть реализовано нескольким способами:

Удаление одной записи:

Процесс удаления - определение элемента в DbSet<T> и вызов метода DbSet<T>.Remove() с передачей экземпляра удаляемого объекта в параметре. Приводит к удалению элемента из коллекции и установке EntityState в Deleted. Удаление из БД - SaveChanges(). Подлежащий удалению элемент должен быть отслеживаемым.
    
Пример:
```csharp
private static void RemoveInventory(int id)
{
    using (var context = new AutoLotEntities())
    {
        var delete = context.Inventories.Find(id);
        if (delete != null)
            context.Inventories.Remove(delete);
        context.SaveChanges();
    }
}
RemoveInventory(1016);
```

Удаление множества записей:

Подлежащие удалению элементы должны быть отслеживаемыми.

Пример:
```csharp
private static void RemoveMultiple()
{
    using (var context = new AutoLotEntities())
    {
        var removeInventories = context.Inventories.Take(2);
        context.Inventories.RemoveRange(removeInventories);
        context.SaveChanges();
    }
}
```

Удаление с использование состояния сущности:

Для удаления создается экземпляр элемента, подлежащего удаленияю, ключу элемента присваивается ключ удаляемого и свойство EntityState устанавливается в Deleted. После вызова SaveChanges() элемент удаляться.

Пример:
```csharp
private static void RemoveInventoryState(int id)
{
    using (var context = new AutoLotEntities())
    {
        Inventory del = new Inventory {CarId = id};
        context.Entry(del).State = EntityState.Deleted;
        try
        {
            context.SaveChanges();
        }
        catch (Exception e)
        {
            Console.WriteLine(e.Message);
        }
    }
}
RemoveInventoryState(1017);
```





