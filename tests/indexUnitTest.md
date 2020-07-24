## Unit-тестирование

Модульное тестирование - тестирование каждой атомарной функции приложения отдельно с использованием объектов искусственно смоделированных средой.

Unit тест должен быть:

- простым в реализации

- автоматизированным и повторяемым

- после написания должен остаться для последующего использования

- кто угодно должнен иметь возможность запусать его

- должен запускаться одним нажатием кнопки

- должен выполняться быстро

- должен быть частью контроля версии

- приложение монолитное - все тесты в директории tests

- приложение компонентное - тесты каждого компонента в директории с компонентом

- тесты могут выноситься в отдельный проект

К каждому проекту - библиотеке следует добавлять свой проект его тестрирования, правила именования:

Проект:

<Project_name>.Core

Его тесты - проект:

<Project_name>.Core.Tests

Именование классов:

Класс:

UserManagement

Тестирующий класс:

UserManagementTests

Именование метода Unit-теста:

[Название метода]_[Сценарий]_[Ожидаемый результат]

Примеры:

Sum_10Plus20_30returned

GetPasswordStrength_AllChars_5Points

Шаблон теста:

Arrange - Переменные, для того, чтобы выполнить тестирование

Act - Действия над системой, для того чтоб произвести тест

Assert - Проверка, что операции завершились успено

## Когда писать Unit тесты:

Для кода:

- сложный код без зависимостей - запутанная бизнес логика и сложные алгоритмы

- не очень сложный код с зависимостями - связующий разные компоненты код

Пример простой, библиотека и ее юнит тест:

```csharp
namespace ClassLibrary1
{
    public class MyCalc
    {
        public int Sum(int a, int b)
        {
            return a + b;
        }
    }
}
namespace ClassLibrary1.Tests
{
    [TestClass]
    public class MyCalcTests
    {
        [TestMethod]
        public void Sum_10Plus20_return30()
        {
            //arrange
            int x = 10;
            int y = 20;
            int expected = 30;

            //act
            MyCalc c = new MyCalc();
            int actual = c.Sum(x, y);

            //assert
            Assert.AreEqual(expected, actual);
        }
    }
}
```

## Инит ресурсов для тестирования

В классе тестрирования можно написать метод (TestInitialize), который будет готовить ресурсы для каждого из выполняемых в классе методов тестирования, а метод (TestCleanup) будет освобождать ресурсы, задействованные первым методом.

```csharp
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
```

В дополнении к этому можно писать СТАТИЧНЫЙ метод (ClassInitialize), готовящий ресурсы перед выполнением всех методов тестирования сразу, а не для каждого, и освобождающий СТАТИЧНЫЙ метод (ClassCleanup).

```csharp
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
```

Атрибуты [AssemplyInitizlize] и [AssemplyCleanup] на классе в проекте тестирования позволяют стоить классы, которые будут инициализировать и удалятьs ресурсы перед выполнением абсолютно всех тестов в проекте тестирования.

## Класс Assert

Класс позволяет различными способами проверять успешность прохождения теста.

Assert - обычные проверки
```csharp
Assert.AreEqual(expected, actual);
```
CollectionAssert - проверки с коллекциями
```csharp
CollectionAssert.AllItemsAreNotNull(employees, "Ошибка NotNull");
```
StringAssert - проверки со строками
```csharp
StringAssert.Contains("Hello World!", "Wor");
```

## Тесты по данным

Класс TestContext служит для определения тесту дополнительных параметров, таких как URI, ASP страницы, источник данных

Допустимо данные для тестов получать из файлов:
```csharp
[DataSource("Microsoft.VisualStudio.TestTools.DataSource.XML",
    "testData.xml",
    "User",
    DataAccessMethod.Sequential)]
[TestMethod]
public void Good_FromXML_Trues()
{
    string name = Convert.ToString(TestContext.DataRow["name"]);
```

## Тестирование методов которые работают с файлами

Для тестирования таких методов используется атрибут DeploymentItem, используемый для определения директорий или файлов, которые нужно скопировать в папку TestResults для доступа к ним во время выполнения тестов:
```csharp
[DeploymentItem("Files\\ExamCreatedResult.txt")]
```
