# Работа EF

## Создание записей данных

Записи добавляются в базу данных после того, как они созданы в коде, добавлены через DbSet<T> и вызван метод SaveChanges(). В этом методе ChangeTracker сообщает, что сущности были добавлены и EF Core создает SQL инструкции добавления записей в базу данных.

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
var newSample = new Sample();
context.Samples.Attach(newSample);
context.SaveChanges();
```

Можно добавлять сразу несколько записей в базу данных использованием метода AddRange().

Пример:

```csharp
var list = new List<Sample>{
    new Sample(),
    new Sample(),
    new Sample(),
};
context.Samples.AddRange(list);
context.SaveChanges();
```

Можно добавляь в базу данных объектные графы - несколько взаимосвязанных объектов, которые являются несколькими записями в разных таблицах базы данных. EF Core создаст связи автоматически.

Пример:

```csharp
var make = new Make { Name = "Make" };
var sample = new Sample { Name = "One" };
((List<Sample>)make.Samples).Add(sample);
context.Makes.Add(make);
context.SaveChanges();
```

Можно добавлять в базу данных в разные таблицы данных связанные записи типа "многие-ко-многим" в удобной и быстрой форме.

Пример:

```csharp
var drivers = new List<Driver>
{
    new() { PersonInfo = new Person { FirstName = "One", LastName = "Two" }},
    new() { PersonInfo = new Person { FirstName = "Three", LastName = "Four" }},
};
var samples = context.Samples.Take(2).ToList();
((List<Driver>)samples[0].Drivers).AddRange(drivers.Take(..1));
((List<Driver>)samples[1].Drivers).AddRange(drivers.Take(1..));
context.SaveChanges();
```

Добавление множества записей в базу данных возможно вызовыми простых методов производного от DbContext класса.

Пример:

```csharp
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
```

## Обновление записей данных

Записи изменяются в базе данных после того, как они будут заргужены через DbSet<T>, изменены и вызван метод SaveChanges(). В этом методе ChangeTracker сообщает обо всех изменениях сущностей и EF Core создает SQL инструкции изменения записей в базе данных.

Когда сущность редактируется, EntityState устанавливается в Modified. После успешного сохранения изменений состояние возвращается к Unchanged.

### Обновление отслеживаемых сущностей

Для обновления нужно загрузить сущность из базы данных, внести в нее изменения и вызвать метод SaveChanges() на производном от DbContext объекте. Не требуется вызывать методы Update() и UpdateRange().

Пример:

```csharp
var sample = context.Samples.First(x => x.Id == 1);
sample.Name = "NewName";
context.SaveChanges();
```

### Обновление неотслеживаемых сущностей

В этом случае сущность создается в коде, а исполняющую среду требуется уведомить о том, что сущность уже существует в базе данных и нуждается в обновлении. Нужно либо вызывать метод Update() на свойстве DbSet<T> производного от DbContext объекта, либо вызывать метод Entry() сразу на производном от DbContext объекте для установки состояния в Modified. В обоих случаях изменения нужно фиксировать методом SaveChanges().

Пример:

```csharp
var sample = context.Samples.AsNoTracking().First(x => x.Id == 1);
var newSample = new Sample
{
    Name = "NewSample", // новое
    Id = sample.Id,
    IsTest = sample.IsTest,
    MakeId = sample.MakeId,
    TimeStamp = sample.TimeStamp,
};
context.Entry(newSample).State = EntityState.Modified;
//context.Update(newSample);
context.SaveChanges();
```

## Удаление записей данных

Запись в таблице базы данных удаляется методом Remove() на свойстве DbSet<T> на производном от DbContext объекте или прямой установкой статуса сущности Deleted. Процесс удаления будет вызывать эффекты каскадирования для навигационных свойств на основе правил, сконфигурированных в методе OnModelCreating().

Когда сущность удаляется, EntityState устанавливается в Deleted. После успешного сохранения изменений сущность исключается из ChangeTracker, перестает отслеживаться, EntityState устанавливается в Detached.

### Удаление отслеживаемых сущностей

Для удаления нужно загрузить сущность из базы данных, установить ее EntityState в Deleted и вызвать метод SaveChanges() на производном от DbContext объекте. Сущность продолжает существовать вне базы данных.

Пример:

```csharp
var sample = context.Samples.First(x => x.Id == 1);
context.Samples.Remove(sample);
context.SaveChanges();
```

### Удаление неотслеживаемых сущностей

Удаление производится вызовом Remove()/RemoveRange() или установкой состояния в Deleted и последующим вызовом SaveChanges(). 

Пример:

```csharp
var sample = context.Samples.AsNoTracking().First(x => x.Id == 1);
context.Entry(sample).State = EntityState.Deleted;
//context.Samples.Remove(sample);
context.SaveChanges();
```






















