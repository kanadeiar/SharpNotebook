# Работа EF

## Создание записей

Записи добавляются в базу данных после того, как они созданы в коде, добавлены через DbSet<T> и вызван метод SaveChanges(). В этом методе ChangeTracker сообщает, что сущности были добавлены и EF Core создает SQL инструкции добавления записей в базу данных.

### EntityState

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
using (var context = new ApplicationDbContext(new DbContextOptions<ApplicationDbContext>()))
{
    var newSample = new Sample();
    context.Samples.Attach(newSample);
    context.SaveChanges();
}
```

Можно добавлять сразу несколько записей в базу данных использованием метода AddRange().

Пример:

```csharp
using (var context = new ApplicationDbContext(new DbContextOptions<ApplicationDbContext>()))
{
    var list = new List<Sample>{
        new Sample(),
        new Sample(),
        new Sample(),
    };
    context.Samples.AddRange(list);
    context.SaveChanges();
}
```

Можно добавляь в базу данных объектные графы - несколько взаимосвязанных объектов, которые являются несколькими записями в разных таблицах базы данных.





### Столбец идентификации

Когда сущность имеет целочисленное свойство определенное как первичный ключ, то такое свойство будет отражатся как первичный ключ SQL Server. EF Core считает, что всякая сущность со значением ключа 0 является новой, а со значением каким-либо - уже существует в базе данных.

Когда нужно добавлять записи с определенными идентфикаторами, то нужно вызывать метод ExecuteSqlRaw() и параметрах метода передавать SQL код деактивации/активации IDENTITY_INSERT.

Пример:

```csharp
context.Database.ExecuteSqlRaw($"SET IDENTITY_INSERT {schema}.{tableName} ON");
//код добавления записей с идентификаторами
context.Database.ExecuteSqlRaw($"SET IDENTITY_INSERT {schema}.{tableName} OFF");
```



971
















