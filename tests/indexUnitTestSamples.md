# Примеры Unit-тестирования

## Примеры инит ресурсов для тестирования

Пример метода инициализации класса (TestInitialize):

```csharp
namespace ClassLibrary1
{
    [Serializable]
    public class MyCalc : IDisposable
    {
        public List<int> Items = new List<int>();

        public void Add(int item)
        {
            Items.Add(item);
        }
        public void Remove(int index)
        {
            Items.RemoveAt(index);
        }
        public int Count { get => Items.Count; }
        public void Dispose()
        {
            //clean class
        }
    }
}
namespace ClassLibrary1.Tests
{
    [TestClass]
    public class TestInitAndClean
    {
        private MyCalc myCalc;
        private int item;
        [TestInitialize]
        public void TestInitialize()
        {
            Debug.WriteLine("TestInitialize");
            item = 10;
            myCalc = new MyCalc();
            myCalc.Add(item);
        }
        [TestCleanup]
        public void TestCleanup()
        {
            Debug.WriteLine("TestCleanup");
            myCalc.Dispose();
        }
        [TestMethod]
        public void MyCalc_Check_ConstainsItem()
        {
            CollectionAssert.Contains(myCalc.Items, item);
        }
        [TestMethod]
        public void MyCalc_RemoveItem_Empty()
        {
            int expected = 0;

            myCalc.Remove(0);

            Assert.AreEqual(expected, myCalc.Count);
        }
    }
}
```

Пример СТАТИЧНОГО метода (ClassInitialize):

```csharp
//тотже класс что и выше
namespace ClassLibrary1.Tests
{
    [TestClass]
    public class ClassInitAndClean
    {
        private static MyCalc myCalc;
        [ClassInitialize]
        public static void ClassInitialize(TestContext context)
        {
            Debug.WriteLine("ClassInitialize");
            myCalc = new MyCalc();
            int item = 10;
            myCalc.Add(item);
        }
        [ClassCleanup]
        public static void ClassCleanup()
        {
            Debug.WriteLine("ClassCleanup");
            myCalc.Dispose();
        }
        [TestMethod]
        public void MyCalc_AddItem()
        {
            int expected = myCalc.Count + 1;

            myCalc.Add(10);

            Assert.AreEqual(expected, myCalc.Count);
        }
        [TestMethod]
        public void MyCalc_DeleteItem()
        {
            int expected = myCalc.Count - 1;

            myCalc.Remove(0);

            Assert.AreEqual(expected, myCalc.Count);
        }
    }
}
```


## Примеры различного применения Assert

Примеры:
```csharp
namespace ClassLibrary1
{
    public class Class1
    {
        public static double GetSqrt(double x)
        {
            return Math.Sqrt(x);
        }
        public string SayHello(string name)
        {
            if (name == null)
                throw new ArgumentNullException("Not Parameter");
            return "Hello " + name;
        }
    }
}
namespace ClassLibrary1.Tests
{
    [TestClass]
    public class Class1Tests
    {
        [TestMethod]
        public void GetSqrt_4_return2()
        {
            const double x = 4.0;
            const double expected = 2.0;

            double actual = Class1.GetSqrt(x);
            //комментарий к ошибке
            Assert.AreEqual(expected, actual, $"Sqrt {x} должне быть = {expected}, а не {actual}");
        }
        [TestMethod]
        public void GetSqrt_10_return3_1_d_0_1()
        {
            const double expected = 3.1;
            const double delta = 0.1;

            double actual = Class1.GetSqrt(10);
            //погрешность в сравнении
            Assert.AreEqual(expected, actual, delta, "fail!");
        }
        [TestMethod]
        public void StringAreEqual()
        {
            const string inp = "hello";
            const string expected = "HELLO";
            //игнорирование регистра
            Assert.AreEqual(expected, inp, true);
        }
        [TestMethod]
        public void StringsAreSame()
        {
            string a = "hello";
            string b = "hello";
            //равенство ссылок
            Assert.AreSame(a, b);
        }
    }
}
namespace ClassLibrary1.Tests
{
    [TestClass]
    public class CollectionTests
    {
        static List<string> employees;
        [ClassInitialize]
        public static void InitializeClass(TestContext context)
        {
            employees = new List<string>();
            employees.Add("Ivan");
            employees.Add("Segey");
            employees.Add("Anton");
            employees.Add("Roman");
        }
        [TestMethod]
        public void TestAreNoNull()
        {
            CollectionAssert.AllItemsAreNotNull(employees, "Ошибка NotNull");
        }
        [TestMethod]
        public void TestAreUnique()
        {
            CollectionAssert.AllItemsAreUnique(employees, "Ошибка уникальности");
        }
        [TestMethod]
        public void TestAreEqual()
        {
            var employees2 = new List<string>();
            employees2.Add("Ivan");
            employees2.Add("Segey");
            employees2.Add("Anton");
            employees2.Add("Roman");
            //равенство коллекций полное
            CollectionAssert.AreEqual(employees2, employees);
        }
        [TestMethod]
        public void TestAreEquivalent()
        {
            var employees2 = new List<string>();
            employees2.Add("Anton");
            employees2.Add("Roman");
            employees2.Add("Ivan");
            employees2.Add("Segey");
            //равенство элементов коллекций, порядок не важен
            CollectionAssert.AreEquivalent(employees2, employees);
        }
        [TestMethod]
        public void TestSubset()
        {
            var employees2 = new List<string>();
            employees2.Add(employees[1]);
            employees2.Add(employees[2]);
            //проверка нахождения элементов первой коллекции во второй
            CollectionAssert.IsSubsetOf(employees2, employees);
        }
    }
}
namespace ClassLibrary1.Tests
{
    [TestClass]
    public class StringTests
    {
        [TestMethod]
        public void TestContains()
        {
            StringAssert.Contains("Hello World!", "Wor");
        }
        [TestMethod]
        public void TestMatches()
        {
            StringAssert.Matches("1234567890",new Regex(@"\d3"));
        }
        [TestMethod]
        public void TestStart()
        {
            StringAssert.StartsWith("Hello World!", "Hello");
        }
        [TestMethod]
        public void TestEnd()
        {
            StringAssert.EndsWith("Hello World!", "World!");
        }
    }
}
namespace ClassLibrary1.Tests
{
    [TestClass]
    public class ExceptionTest
    {
        [ExpectedException(typeof(ArgumentNullException), "Исключение не посылается в ответ на null!")]
        [TestMethod]
        public void SayHello_Null_Exception()
        {
            Class1 class1 = new Class1();
            class1.SayHello(null);
        }
        [TestMethod]
        public void SayHello_And_HelloAnd()
        {
            Class1 class1 = new Class1();
            const string expected = "Hello And";

            string actual = class1.SayHello("And");

            StringAssert.Equals(expected, actual);
        }
    }
}
```

## Пример теста по данным из файла и обычным жестко закодированным данным

Пример файла данных:
```csharp
<?xml version="1.0" encoding="utf-8"?>
<UserDetails>
  <User name="Dmitriy" age="18"/>
  <User name="Andrei" age="20"/>
  <User name="Next" age="22"/>
</UserDetails>
```

Пример теста:
```csharp
namespace ClassLibrary1
{
    public class Class1
    {
        public bool Good(string name, int age)
        {
            if (name.Length < 3)
                return false;
            if (age <= 0)
                return true;
            return true;
        }
    }
}
namespace ClassLibrary1.Tests
{
    [TestClass]
    public class Class1Tests
    {
        public TestContext TestContext { get; set; }
        private Class1 class1 = new Class1();
     
        [DataTestMethod] //жестко закодированные данные
        [DataRow("Andrei", 18)]
        [DataRow("Maxim", 20)]
        [DataRow("Xianon", 22)]
        public void GoodDataTest_Data_Trues(string name, int age)
        {
            Assert.IsTrue(class1.Good(name, age));
        }

        [DataSource("Microsoft.VisualStudio.TestTools.DataSource.XML",
            "testData.xml",
            "User",
            DataAccessMethod.Sequential)]
        [TestMethod]
        public void Good_FromXML_Trues()
        {
            string name = Convert.ToString(TestContext.DataRow["name"]);
            int age = Convert.ToInt32(TestContext.DataRow["age"]);

            bool result = class1.Good(name, age);

            Assert.IsTrue(result);
        }
    }
}
```

## Пример тестирования методов которые работают с файлами

Пример файла шаблона в папке Files:
```csharp
<div>
	Экзамен {Name} добавлен в каталог экзаменов. <br />
	Уровень сложности экзамена {Level} <br />
</div>
```
Пример файла заполненного шаблона в папке Files:
```csharp
<div>
	Экзамен TestExam добавлен в каталог экзаменов. <br />
	Уровень сложности экзамена Beginer <br />
</div>
```

Пример:
```csharp
namespace ClassLibrary1
{
    public class Class1
    {
        public string TemplateFolder { get; set; }
        public string FromTemplate(string name, string level)
        {
            string path = Path.Combine(TemplateFolder, "ExamCreatedTemplate.txt");
            string template = File.ReadAllText(path);
            template = template.Replace("{Name}", name);
            template = template.Replace("{Level}", level);
            return template;
        }
    }
}
namespace ClassLibrary1.Tests
{
    [TestClass]
    public class Class1Tests
    {
        [TestMethod]
        [DeploymentItem("Files\\ExamCreatedResult.txt")]
        [DeploymentItem("Files\\ExamCreatedTemplate.txt")]
        public void FromTemplate_PassTestValue_StringFromTemplateReturned()
        {
            Class1 class0 = new Class1();
            class0.TemplateFolder = "."; //текущая директория

            string expected = File.ReadAllText("ExamCreatedResult.txt");

            string actual = class0.FromTemplate("TestExam", "Beginer");

            Assert.AreEqual(expected, actual);
        }
    }
}
```

