# Тестирование c инверсией управления

Внешняя зависимость это объект, с которым взаимодействует код, над которым нет прямого контроля. (Т.е. файловая система, потоки, память, службы и т.д.) Для борьбы с внешними зависимостями используются заглушки (stubs).

Для того, чтоб избежать прямой зависимости от, например, файловой системы, вводится дополнительный объект, в который переносится весь код работы с файловой системой. Такая структура позволяет в дальнейшем заменить этот новый объект на подставной и выполнить тестирование основого кода без внешних зависимостей.

Инверсия управления (Inversion of Control) - важный абстрактный принцип, являющий собой набор рекомендаций по написани слабо связанного кода. Каждый компонент системы должен быть как можно более изолированным от других компонентов и не полагаться в своей работе на детали точной реализации других объектов.

Dependency Injection - паттерн описывающий технику внедрения внешних зависимостей в программный компонент. Способы внедрения:

- через конструктор.

- через свойство.

- через интерфейс.

Возможно создание экзепляров зависимостей через контейнеры.

## Тестирование класса с внешней зависимостью внедрением через конструктор

В конструктор передается заглушка.

Пример:
```csharp
// тестируемый класс и приложение
// InternalsVisibleTo - указывает, что типы, которые обычно доступны только в текущей сборке будут доступны также в сборке,
// которая определена в параметрах.
[assembly: InternalsVisibleTo("ClassLibrary1.Tests")]
namespace ConsoleApp1
{
    class FileManager
    {
        IDataAccess dataAccess; //уменьшение связанности применением IDataAccess
        public FileManager() //стандартный конструктор
        {
            dataAccess = new FileDataObject();
        }
        public FileManager(IDataAccess dataAccess) //тестировочный конструктор
        {
            this.dataAccess = dataAccess;
        }
        /// <summary> Поиск файла </summary>
        public bool FindFile(string filename)
        {
            List<string> files = dataAccess.GetFiles();
            foreach (var file in files)
            {
                if (file.Contains(filename))
                {
                    return true;
                }
            }
            return false;
        }
    }
    class Program
    {
        static void Main(string[] args)
        {
            string filename = "sample.txt";

            FileManager manager = new FileManager();

            if (manager.FindFile(filename))
                Console.WriteLine("Файл найден!");
            else
                Console.WriteLine("Файл не найден!");

            Console.ReadKey();
        }
    }
}
//вспомогательные классы в каталоге отдельном
namespace ConsoleApp1.DataAccessObjects
{
    /// <summary> Стандартное получение списка файлов в каталоге </summary>
    class FileDataObject : IDataAccess
    {
        public List<string> GetFiles()
        {
            string path = Directory.GetCurrentDirectory();
            List<string> list = new List<string>();
            DirectoryInfo d = new DirectoryInfo(path);
            FileInfo[] files = d.GetFiles();
            foreach (var file in files)
            {
                list.Add(file.Name);
            }
            return list;
        }
    }
    public interface IDataAccess
    {
        List<string> GetFiles();
    }
    /// <summary> Объект-заглушка, фиктивное получение файлов из каталога </summary>
    public class StubFileDataObject : IDataAccess
    {
        public List<string> GetFiles()
        {
            List<string> list = new List<string>();
            list.Add("file1.txt");
            list.Add("file2.log");
            list.Add("file3.exe");
            list.Add("main.log");
            return list;
        }
    }
}
//тестирующий класс
namespace ConsoleApp1.Tests
{
    [TestClass]
    public class FileManagerTests
    {
        [TestMethod]
        public void FindFile_DI_True()
        {
            IDataAccess access = new StubFileDataObject();
            FileManager fileManager = new FileManager(access);
            string filename = "file1.txt";

            bool actual = fileManager.FindFile(filename);

            Assert.IsTrue(actual);
        }
    }
}
```

## Тестирование класса с внешней зависимостью внедрением через свойство

В свойство тестируемого класса может записываться заглушка

Пример:
```csharp
// тестируемый класс и приложение
// InternalsVisibleTo - указывает, что типы, которые обычно доступны только в текущей сборке будут доступны также в сборке,
// которая определена в параметрах.
[assembly: InternalsVisibleTo("ClassLibrary1.Tests")]
namespace ConsoleApp1
{
    class FileManager
    {
        IDataAccess dataAccess; //уменьшение связанности применением IDataAccess
        public FileManager()
        {
        }
        /// <summary> Свойство внедрения зависимости </summary>
        public IDataAccess DataAccess
        {
            set => dataAccess = value;
            get
            {
                if (dataAccess == null)
                    throw new MemberAccessException("dataAccess не инициализирован");
                return dataAccess;
            }
        }
        /// <summary> Поиск файла </summary>
        public bool FindFile(string filename)
        {
            List<string> files = dataAccess.GetFiles();
            foreach (var file in files)
            {
                if (file.Contains(filename))
                {
                    return true;
                }
            }
            return false;
        }
    }
}
namespace ConsoleApp1
{
    class Program
    {
        static void Main(string[] args)
        {
            string filename = "sample.txt";

            FileManager manager = new FileManager();

            manager.DataAccess = new FileDataObject();

            if (manager.FindFile(filename))
                Console.WriteLine("Файл найден!");
            else
                Console.WriteLine("Файл не найден!");

            Console.ReadKey();
        }
    }
}
//вспомогательные классы теже что и выше
//тестирующий класс
namespace ConsoleApp1.Tests
{
    [TestClass]
    public class FileManagerTests
    {
        [TestMethod]
        public void FindFile_DI_True()
        {
            IDataAccess access = new StubFileDataObject();
            FileManager fileManager = new FileManager();
            fileManager.DataAccess = access; //внедрение зависимости
            string filename = "file1.txt";

            bool actual = fileManager.FindFile(filename);

            Assert.IsTrue(actual);
        }
    }
}
```

## Тестирование класса с внешней зависимостью внедрением через интерфейс

В метод тестируемого класса может записываться через интерфейс класс-заглушка

```csharp
// тестируемый класс и приложение
// InternalsVisibleTo - указывает, что типы, которые обычно доступны только в текущей сборке будут доступны также в сборке,
// которая определена в параметрах.
[assembly: InternalsVisibleTo("ClassLibrary1.Tests")]
namespace ConsoleApp1
{
    class FileManager
    {
        /// <summary> Поиск файла </summary>
        public bool FindFile(string filename, IDataAccess dataAccess) //внедрение зависимости через интерфейс
        {
            List<string> files = dataAccess.GetFiles();
            foreach (var file in files)
            {
                if (file.Contains(filename))
                {
                    return true;
                }
            }
            return false;
        }
    }
}
namespace ConsoleApp1
{
    class Program
    {
        static void Main(string[] args)
        {
            string filename = "sample.txt";

            FileManager manager = new FileManager();

            if (manager.FindFile(filename, new FileDataObject()))
                Console.WriteLine("Файл найден!");
            else
                Console.WriteLine("Файл не найден!");

            Console.ReadKey();
        }
    }
}
//вспомогательные классы теже что и выше
//тестирующий класс
namespace ConsoleApp1.Tests
{
    [TestClass]
    public class FileManagerTests
    {
        [TestMethod]
        public void FindFile_DI_True()
        {
            IDataAccess access = new StubFileDataObject();
            FileManager fileManager = new FileManager();
            string filename = "file1.txt";

            bool actual = fileManager.FindFile(filename, access);

            Assert.IsTrue(actual);
        }
    }
}
```

## Тестирование класса с внешней зависимостью внедрением через фабрику

Класс фабрика заменяет стандартный класс на заглушку.

Пример:
```csharp
//класс-фабрика:
namespace ConsoleApp1.DataAccessObjects
{
    public class DataAccessFactory
    {
        private static IDataAccess dataAccess;
        /// <summary> Создание стандартного или заглушного класса </summary>
        internal static IDataAccess Create()
        {
            if (dataAccess != null)
                return dataAccess;
            return new FileDataObject();
        }
        [Conditional("DEBUG")]
        public static void SetDataAccessObject(IDataAccess customDataAccess)
        {
            dataAccess = customDataAccess;
        }
    }
}
// тестируемый класс и приложение
// InternalsVisibleTo - указывает, что типы, которые обычно доступны только в текущей сборке будут доступны также в сборке,
// которая определена в параметрах.
[assembly: InternalsVisibleTo("ClassLibrary1.Tests")]
namespace ConsoleApp1
{
    class FileManager
    {
        private IDataAccess dataAccess;

        public FileManager()
        {
            dataAccess = DataAccessFactory.Create();
        }

        /// <summary> Поиск файла </summary>
        public bool FindFile(string filename) //внедрение зависимости через интерфейс
        {
            List<string> files = dataAccess.GetFiles();
            foreach (var file in files)
            {
                if (file.Contains(filename))
                {
                    return true;
                }
            }
            return false;
        }
    }
}
namespace ConsoleApp1
{
    class Program
    {
        static void Main(string[] args)
        {
            string filename = "sample.txt";

            FileManager manager = new FileManager();

            if (manager.FindFile(filename))
                Console.WriteLine("Файл найден!");
            else
                Console.WriteLine("Файл не найден!");

            Console.ReadKey();
        }
    }
}
//тестирование класса
namespace ConsoleApp1.Tests
{
    [TestClass]
    public class FileManagerTests
    {
        [TestMethod]
        public void FindFile_DI_True()
        {
            DataAccessFactory.SetDataAccessObject(new StubFileDataObject());
            FileManager fileManager = new FileManager();
            string filename = "file1.txt";

            bool actual = fileManager.FindFile(filename);

            Assert.IsTrue(actual);
        }
    }
}
```

## Тестирование класса с внешней зависимостью внедрением через переопределяемый фабричный метод в унаследованном классе

```csharp
// тестируемый класс и приложение
// InternalsVisibleTo - указывает, что типы, которые обычно доступны только в текущей сборке будут доступны также в сборке,
// которая определена в параметрах.
[assembly: InternalsVisibleTo("ClassLibrary1.Tests")]
namespace ConsoleApp1
{
    class FileManager
    {
        public FileManager()
        {
        }
        protected virtual IDataAccess CreateDataAccess() //доступный для переопределения фабричный метод
        {
            return new FileDataObject();
        }
        /// <summary> Поиск файла </summary>
        public bool FindFile(string filename) //внедрение зависимости через интерфейс
        {
            IDataAccess dataAccess = CreateDataAccess(); //создание фабричным методом
            List<string> files = dataAccess.GetFiles();
            foreach (var file in files)
            {
                if (file.Contains(filename))
                {
                    return true;
                }
            }
            return false;
        }
    }
}
namespace ConsoleApp1
{
    class Program
    {
        static void Main(string[] args)
        {
            string filename = "sample.txt";
            
            FileManager manager = new FileManager();

            if (manager.FindFile(filename))
                Console.WriteLine("Файл найден!");
            else
                Console.WriteLine("Файл не найден!");

            Console.ReadKey();
        }
    }
}
//остальное тоже самое
//тестирование класса
namespace ConsoleApp1.Tests
{
    class FileManagerOverTest : FileManager
    {
        protected override IDataAccess CreateDataAccess()
        {
            return new StubFileDataObject(); //переопределение на заглушку
        }
    }
    [TestClass]
    public class FileManagerTests
    {
        [TestMethod]
        public void FindFile_DI_True()
        {
            FileManager fileManager = new FileManagerOverTest();
            string filename = "file1.txt";

            bool actual = fileManager.FindFile(filename);

            Assert.IsTrue(actual);
        }
    }
}
```




