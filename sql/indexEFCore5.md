# NET EF Core 5

## Аннотации данных EF

Аннотация данных         | Описание
-------------------------|-------------------------
Table                    | Описание таблицы и сущности для сущности
Column                   | Значение колонки для свойства модели
Key                      | Главный ключ модели, включает также аннотацию [Required]
Required                 | Свойство не может быть нулевым в модели
ForeignKey               | Свойство, которое должно быть внешним ключом навигационного свойства модели
InverseProperty          | Объявляет навигационное свойство на другом конце отношений
StringLength             | Максимальная длинна строкового свойства модели
TimeStamp                | Тип строки, опредяющей проверку конкуренктного доступа к записи при производстве действий с сущностью
ConcurrencyCheck         | Флаговое поле должно быть использовано при проверке конкуренктного доступа
DatabaseGenerated        | Спецификация как поле будет генерироваться базой данных - вычисляемое, идентификатор или отсутствует
DataType                 | Описание более специфичного типа поля
NotMapped                | Исключение свойства в классе из учета, не будет создано в базе данных

## Контекст базы данных

Должен быть унаследован от DbContext. 

Главные члены:

Компонент                | Описание
-------------------------|-------------------------
Database                 | Описывает доступ к зависимым от базы данных информации и функциональности + сырой SQL
Model                    | Данные по форме сущности, отношения между ними и как они проецируются на базу данных.
ChangeTracker            | Предоставляет доступ к информации и операциям для инстансов сущностей этого контекста
DBSet<T>                 | Используется для запросов и сохранения массивов сущностей приложения. LINQ транслируется прямо в код SQL.
Entry, Entry<Entity>     | Предоставляет доступ к отслеживанию сущностей, измению их статуса. Также может быть вызвано на сущности не отслеживаемой для установки отслеживания.
SaveChanges, SaveChangesAsync | Сохранение всех воздействий на базу данных в базе данных и возвращает число затронутых сущностей.
OnConfiguring            | Используется для создания и модификации опций контекста. Исполняется каждый раз как создается контекст. Желательно использовать DbContextOptions во время выполнения и IDesignTimeDbContextFactory во время разработки.
OnModelCreating          | Вызывается когда модель инициализирована, но перед финализацией.

Библиотека доступа к данным.

Методы Database предназначенные для удаления и воссоздания базы данных:

Методы                   | Описание
-------------------------|-------------------------
EnsureDeleted()          | Удаляет базу данных, если она существует
EnsuteCreated()          | Создает базу данных на основе файла <...>ModelSnapshot.cs
Migrate()                | Создает базу данных и выполняет все миграции, если они сущетвует. 

## Библиотека моделей доступа к данным - "Students.Dal.Models".

Модели данных:
```csharp
namespace Students.Dal.Models.Base
{
    public abstract class Entity
    {
        public int Id { get; set; }
        [Timestamp]
        public byte[] Timestamp { get; set; }
    }
}
namespace Students.Dal.Models
{
    public partial class Student : Entity
    {
        [Required, MaxLength(100)]
        public string LastName { get; set; }
        [Required, MaxLength(100)]
        public string FirstName { get; set; }
        [Required, MaxLength(100)]
        public string Patronymic { get; set; }
        public DateTime Birthday { get; set; }
        public float Rating { get; set; }
        [MaxLength(50)]
        public string Pet { get; set; }
        public virtual ICollection<Course> Courses { get; set; } = new List<Course>();
        public Group Group { get; set; }
    }
    [MetadataType(typeof(StudentMetaData))]
    public partial class Student
    {
        public override string ToString()
        {
            return $"{FullName} питомец: {this.Pet ?? "Без питомца"}";
        }
        [NotMapped]
        public string FullName => $"{LastName} {FirstName} {Patronymic}";
    }
    [Table("StarStudents")]
    public class StarStudent : Student
    {
        public int Star { get; set; }
        public override string ToString()
        {
            return $"Особый студент {FullName} питомец: {this.Pet ?? "Без питомца"} звезность: {Star}";
        }
    }
    public class Group : Entity
    {
        [Required, StringLength(100)]
        public string Name { get; set; }
        public virtual ICollection<Student> Students { get; set; } = new List<Student>();
    }
    public class Course : Entity
    {
        [Required, StringLength(100)]
        public string Name { get; set; }
        public string Description { get; set; }
        public virtual ICollection<Student> Students { get; set; } = new List<Student>();
    }
    public class StudentMetaData
    {
        [Display(Name = "Pet Name")]
        public string Pet;
        [StringLength(100, ErrorMessage = "Вводите фамилию длинной меньше 100 символов")]
        public string LastName;
        [StringLength(100, ErrorMessage = "Вводите имя длинной меньше 100 символов")]
        public string FirstName;
        [StringLength(100, ErrorMessage = "Вводите отчество длинной меньше 100 символов")]
        public string Patronymic;
    }
}
```

## Библиотека доступа к данным - "Students.Dal".

Пакеты:

- Microsoft.EntityFrameworkCore.SqlServer

- Microsoft.EntityFrameworkCore.Tools

Для включения управления с помощью консоли (устаревшее) (на консольном приложении):
```csharp
<ItemGroup>
<DotNetCliToolReference Include="Microsoft.EntityFrameworkCore.Tools.DotNet" Version="2.0.0" />
</ItemGroup>
```
Новое (на библиотеки доступа к данным), установка консолного управления:

dotnet new tool-manifest

dotnet tool install dotnet-ef

Команды: dotnet ef,    dotnet ef migrations add Initial,      dotnet ef database update,        dotnet ef database drop, 

dotnet ef migrations add InitialCreate

Фабрика времени разработки:
```csharp
namespace Students.Dal
{
    public class StudentsContextFactory : IDesignTimeDbContextFactory<StudentsContext>
    {
        public StudentsContext CreateDbContext(string[] args)
        {
            var optionsBuilder = new DbContextOptionsBuilder<StudentsContext>();
            var connectionString = @"Data Source=(localdb)\MSSQLLocalDB;Initial Catalog=StudentsDev.DB; "+
                                   "Integrated security=True; MultipleActiveResultSets=True; App=EntityFramework;";
            optionsBuilder.UseSqlServer(connectionString, options => options.EnableRetryOnFailure());
            return new StudentsContext(optionsBuilder.Options);
        }
    }
}
```

Контекст базы данных:
```csharp
namespace StudentsDAL.EF
{
    public class StudentsContext : DbContext
    {
        public DbSet<Course> Courses { get; set; }
        public DbSet<Group> Groups { get; set; }
        public DbSet<Student> Students { get; set; }
        public DbSet<StarStudent> StarStudents { get; set; }
        public StudentsContext(DbContextOptions options) : base(options)
        {
        }
        internal StudentsContext()
        {
        }
        protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        {
            if (!optionsBuilder.IsConfigured)
            {
                var connectionString = @"Data Source=(localdb)\MSSQLLocalDB;Initial Catalog=StudentsDev.DB; "+
                                       "Integrated security=True; MultipleActiveResultSets=True; App=EntityFramework;";
                optionsBuilder.UseSqlServer(connectionString, options => options.EnableRetryOnFailure());
            }
        }
    }
}
```

Инициализатор данных:
```csharp
namespace Students.Dal.DataInit
{
    public class StudentsDataInit
    {
        private static Random rnd = new Random();
        public static void RecreateDatabase(StudentsContext context)
        {
            context.Database.EnsureDeleted();
            context.Database.Migrate();
        }
        public static void InitData(StudentsContext context)
        {
            var cources = Enumerable.Range(1, 10).Select(c => new Course
            {
                Name = $"Учебный предмет №{rnd.Next(20)}",
                Description = $"Описание предмета {c}",
            }).ToList();
            context.Courses.AddRange(cources);
            var groups = Enumerable.Range(1, 10).Select(c => new Group
            {
                Name = $"Группа{rnd.Next(100, 900)}",
            }).ToList();
            context.Groups.AddRange(groups);
            var students = Enumerable.Range(1, 30).Select(s => new Student
            {
                LastName = $"Иванов_{s}",
                FirstName = $"Иван_{s + 1}",
                Patronymic = $"Иванович_{s - 1}",
                Birthday = new DateTime(rnd.Next(1930, 1980), 1,1),
                Rating = (float) rnd.NextDouble() * 10.0f,
                Pet = (s % 2 == 1) ? $"Кошка{s}" : String.Empty,
                Courses = Enumerable.Range(1, rnd.Next(8))
                    .Select(c => cources[rnd.Next(cources.Count)]).ToList(),
                Group = groups[rnd.Next(groups.Count)],
            }).ToList();
            context.Students.AddRange(students);
            context.StarStudents.Add(new StarStudent
            {
                LastName = "Петров",
                FirstName = "Петр",
                Patronymic = "Петрович",
                Birthday = new DateTime(1900, 1, 1),
                Rating = 10,
                Pet = "Лев",
                Courses = Enumerable.Range(1, rnd.Next(8))
                    .Select(c => cources[rnd.Next(cources.Count)]).ToList(),
                Group = groups[rnd.Next(groups.Count)],
                Star = 5,
            });
            context.Database.OpenConnection();
            try
            {
                context.SaveChanges();
            }
            catch (Exception ex)
            {
                Console.WriteLine($"Ошибка доступа к базе данных: {ex.Message}");
            }
            finally
            {
                context.Database.CloseConnection();
            }
        }
        public static void ClearData(StudentsContext context)
        {
            ExecuteDeleteSql(context, "CourseStudent");
            ExecuteDeleteSql(context, "StarStudents");
            ExecuteDeleteSql(context, "Students");
            ExecuteDeleteSql(context, "Groups");
            ExecuteDeleteSql(context, "Courses");
            ResetIdentity(context);
            static void ExecuteDeleteSql(StudentsContext context, string tableName)
            {
                var rawSqlString = $"Delete from dbo.{tableName}";
                context.Database.ExecuteSqlRaw(rawSqlString);
            }
            static void ResetIdentity(StudentsContext context)
            {
                var tables = new[] { "Students", "Groups", "Courses" };
                foreach (var item in tables)
                {
                    var rawSqlString = $"DBCC CHECKIDENT (\"dbo.{item}\", RESEED, -1);";
                    context.Database.ExecuteSqlRaw(rawSqlString);
                }
            }
        }
    }
}
```

Для доступности класса консольному приложению - файл "AssemblyInfo.cs":
```csharp
using System.Runtime.CompilerServices;
[assembly: InternalsVisibleTo("StudentsDAL.TestDriver")]
```

Создание миграции, обновление базы данных (На запуск поставить проект где стоит ef tools, консоль включить на проекте в котором контекст базы данных и фабрика):

Add-Migration init,       Update-Database,      Get-Migrations,     Drop-Migration,       Drop-Database

## Репозиторий

Универсальный репозиторий базы данных
```csharp
namespace Students.Dal.Repos.Base
{
    public interface IRepo<T>
    {
        int Add(T Entity, bool Persist = true);
        int AddRange(IEnumerable<T> Entites, bool Persist = true);
        int Update(T Entity, bool Persist = true);
        int UpdateRange(IEnumerable<T> Entites, bool Persist = true);
        int Delete(T Entity, bool Persist = true);
        int Delete(int Id, byte[] Timestamp, bool Persist = true);
        int DeleteRange(IEnumerable<T> Entites, bool Persist = true);
        T Find(int? Id);
        IEnumerable<T> Get(Expression<Func<T, bool>> Where);
        IEnumerable<T> Get<TSortField>(Expression<Func<T, bool>> Where, Expression<Func<T, TSortField>> OrderBy, bool Ansc);
        IEnumerable<T> GetAll();
        IEnumerable<T> GetAll<TSortField>(Expression<Func<T, TSortField>> OrderBy, bool Ansc);
        int SaveChanges();
    }
}
namespace Students.Dal.Repos.Base
{
    public class BaseRepo<T> : IDisposable, IRepo<T> where T : Entity, new()
    {
        private bool _IsDisposed;
        private readonly bool _DisposeContext;
        public DbSet<T> Table { get; }
        public StudentsContext Context { get; }
        public BaseRepo(StudentsContext context)
        {
            Context = context;
            Table = Context.Set<T>();
        }
        public BaseRepo(DbContextOptions<StudentsContext> options) : this(new StudentsContext(options))
        {
            _DisposeContext = true;
        }
        public void Dispose()
        {
            Dispose(true);
            GC.SuppressFinalize(this);
        }
        protected virtual void Dispose(bool disposing)
        {
            if (_IsDisposed)
                return;
            if (disposing)
                if (_DisposeContext)
                    Context.Dispose();
            _IsDisposed = true;
        }
        ~BaseRepo() => Dispose(false);


        public int Add(T Entity, bool Persist = true)
        {
            Table.Add(Entity);
            return Persist ? SaveChanges() : 0;
        }
        public int AddRange(IEnumerable<T> Entites, bool Persist = true)
        {
            Table.AddRange(Entites);
            return Persist ? SaveChanges() : 0;
        }
        public int Update(T Entity, bool Persist = true)
        {
            Table.Update(Entity);
            return Persist ? SaveChanges() : 0;
        }
        public int UpdateRange(IEnumerable<T> Entites, bool Persist = true)
        {
            Table.UpdateRange(Entites);
            return Persist ? SaveChanges() : 0;
        }
        public int Delete(T Entity, bool Persist = true)
        {
            Table.Remove(Entity);
            return Persist ? SaveChanges() : 0;
        }
        public int Delete(int Id, byte[] Timestamp, bool Persist = true)
        {
            Context.Entry(new T {Id = Id, Timestamp = Timestamp}).State = EntityState.Deleted;
            return Persist ? SaveChanges() : 0;
        }
        public int DeleteRange(IEnumerable<T> Entites, bool Persist = true)
        {
            Table.RemoveRange(Entites);
            return Persist ? SaveChanges() : 0;
        }
        public T Find(int? Id) => Table.Find(Id);
        public IEnumerable<T> Get(Expression<Func<T, bool>> Where) => Table.Where(Where);
        public IEnumerable<T> Get<TSortField>(Expression<Func<T, bool>> Where, Expression<Func<T, TSortField>> OrderBy, bool Ansc) => 
            (Ansc ? Table.Where(Where).OrderBy(OrderBy) : Table.Where(Where).OrderByDescending(OrderBy));
        public IEnumerable<T> GetAll() => Table;
        public IEnumerable<T> GetAll<TSortField>(Expression<Func<T, TSortField>> OrderBy, bool Ansc) => 
            (Ansc ? Table.OrderBy(OrderBy) : Table.OrderByDescending(OrderBy));
        public int SaveChanges()
        {
            try
            {
                return Context.SaveChanges();
            }
            catch (DbUpdateConcurrencyException ex)
            {
                throw;
            }
            catch (RetryLimitExceededException ex)
            {
                throw;
            }
            catch (DbUpdateException ex)
            {
                throw;
            }
            catch (Exception ex)
            {
                throw;
            }
        }
    }
}
namespace StudentsDAL.Repos.Interfaces
{
    public interface IStudentsRepo : IRepo<Student>
    {
    }
}
namespace StudentsDAL.Repos
{
    public class StudentsRepo : BaseRepo<Student>, IStudentsRepo
    {
        public new IEnumerable<Student> GetAll() => GetAll(x => x.LastName, false);
        public IEnumerable<Student> GetTopRating() => Get(x => x.Rating > 2.0f);
        public List<Student> Search(string SearchString)
        {
            return Context.Students.Where(c => Functions.Like(c.Pet, $"")).ToList();
        }
        public StudentsRepo() : base(new StudentsContext())
        {
        }
        public StudentsRepo(StudentsContext context) : base(context)
        {
        }
        public StudentsRepo(DbContextOptions<StudentsContext> options) : base(options)
        {
        }
        public List<Student> GetRelatedData() => Context.Students.FromSqlRaw("SELECT * FROM Students")
            .Include(x => x.Group).Include(x => x.Courses).ToList();
    }
}
```

## Консольное приложение разработки библиотеки - "Students.Dal.TestDriver".
Приложение тестирования базы данных

```csharp
namespace StudentsDAL.TestDriver
{
    class Program
    {
        static void Main(string[] args)
        {
            ConsoleToRussian();
            Console.WriteLine("Тестирование библиотеки доступа к данным");
            using (var context = new StudentsContext())
            {
                //StudentsDataInit.RecreateDatabase(context);
                //StudentsDataInit.ClearData(context);
                StudentsDataInit.InitData(context);
            }
            using (var repo = new StudentsRepo())
            {
                Console.WriteLine("Студенты:");
                foreach (var student in repo.GetAll())
                {
                    Console.WriteLine(student);
                }
            }
            Console.WriteLine("Нажмите любую кнопку ...");
            Console.ReadKey();
        }
        private static void ConsoleToRussian()
        {
            [DllImport("kernel32.dll")] static extern bool SetConsoleCP(uint pagenum);
            [DllImport("kernel32.dll")] static extern bool SetConsoleOutputCP(uint pagenum);
            SetConsoleCP(65001);        //установка кодовой страницы utf-8 (Unicode) для вводного потока
            SetConsoleOutputCP(65001);  //установка кодовой страницы utf-8 (Unicode) для выводного потока
        }
    }
}
```


