# Рефлексия (динамическая загрузка сборок, позднее связывание, атрибуты, расширяемое приложение)

В процессе работы приложения может возникнуть потребность в выявлении типов во время выполнения приложения. Каждая сборка .NET содержит описание набора используемых классов, интерфесов, методов, свойств, и пр. 

Рефлексия "самопознание" позволяет определить составные элементы сборки. Это процесс обнаружения типов во время работы программы, используя дружественную объектную модель. Можно также получать набор интерфейсов этого типа, параметры метода и другие детали.

Основные члены пространства рефлексии Object.Reflexion:
```csharp
Assembly        Класс содержит несколько членов, которые позволяют загружать, исследовать и манипулировать сборкой
AssembleName    Идентичность сборки
EventInfo       Информация о заданном событии
FieldInfo       Информация о заданном поле
MemberInfo      Определяет общее поведение типов для остальных членов
MethodInfo      Информация о заданном методе
Module          Доступ к заданному модулю внутри многофайловой сборки
ParameterInfo   Информация о заданном параметре
PropertyInfo    Информация о заданном свойстве
```

Исследование метаданных типа с помощью System.Type:
```csharp
IsAbstract, IsArray, IsClass, IsCOMObject, IsEnum, IsGenericTypeDefifnition, IdGenericParameter, IsInterface, IsPrimitive, IsNestedPrivate, IsNestedPublic, IsSealed, IsValueType - выяснение базовых характеристик типа
GetConstructor, GetEvents, GetFields, GetInterfaces, GetMembers, GetMethods, GetNestedTypes, GetProperities - получение массива интересующих элементов.
FindMembers - массив объектов MemberInfo на основе указанного критерия поиска
GetType - статический метод возвращает экземпляр Type с заданным строковым имененм
InvokeMember - выполнение "поздего связывания" для указанного элемента
```

## Получение типа
```csharp
Type type = typeof(MyClass);
```
В роли аргумента может выступать имя класса - как встроенного, так и созданного пользователем.
Вертаемый тип - Type.

```csharp
MyClass variable = new MyClass();
Type type = variable.GetType();
```
В роли аргумента здесь выступает экземпляр класса, результат будет такой-же.
Вертаемый тип - Type.

Еще пример использования:
```csharp
//из внешней сборки
Type t1 = Type.GetType("MyLib.Person, MyLib");
//о типе вложенного в тип (+)
Type t2 = Type.GetType("MyLib.Person+TypeWork");
```

## Просмотр метаданных

Рефлексия методов:
```csharp
public static void Main()
{
    var t1 = typeof(int);
    ListMethods(t1);
    Console.ReadLine();
}
static void ListMethods(Type t)
{
    //MethodInfo[] mis = t.GetMethods();
    var mis = t.GetMethods().Select(m => m.Name);
    foreach (var m in mis)
        Console.WriteLine($"->{m}");
}
```
Рефлексия полей и свойств:
```csharp
public static void Main()
{
    var t1 = typeof(string);
    ListProps(t1);
    Console.ReadLine();
}
static void ListProps(Type t)
{
    var propNames = t.GetProperties().Select(p => p.Name);
    foreach (var p in propNames)
        Console.WriteLine($"-> {p}");
}
```
Рефлексия реализованных интерфейсов:
```csharp
public static void Main()
{
    var t1 = typeof(string);
    ListInterfaces(t1);
    Console.ReadLine();
}
static void ListInterfaces(Type t)
{
    var ifaces = t.GetInterfaces().Select(i => i.Name);
    foreach (var i in ifaces)
        Console.WriteLine(i);
}
```

Рефлексия разнообразных деталей:
```csharp
public static void Main()
{
    var t1 = typeof(string);
    Console.WriteLine($"Базовый класс: {t1.BaseType}");
    Console.WriteLine($"Абстрактный класс: {t1.IsAbstract}");
    Console.WriteLine($"Запечатанный класс: {t1.IsSealed}");
    Console.WriteLine($"Обобщенный: {t1.IsGenericTypeDefinition}");
    Console.WriteLine($"Класс: {t1.IsClass}");
    Console.ReadLine();
}
```

Пример приложения работы с рефлексией:
```csharp
public static void Main()
{
    while (true)
    {
        Console.WriteLine("Введите имя типа (q-выход):");
        string typeName = Console.ReadLine();
        if (typeName.Equals("Q", StringComparison.OrdinalIgnoreCase))
            break;
        var t = Type.GetType(typeName);
        if (t == null)
        {
            Console.WriteLine("Не удается найти такой тип");
            continue;
        }
        var mis = t.GetMethods().Select(m => m.Name);
        foreach (var m in mis)
            Console.WriteLine($"->{m}");
        var propNames = t.GetProperties().Select(p => p.Name);
        foreach (var p in propNames)
            Console.WriteLine($"-> {p}");
        var ifaces = t?.GetInterfaces().Select(i => i.Name);
        foreach (var i in ifaces)
            Console.WriteLine(i);
    }
    Console.ReadLine();
}
```

Пример рефлексии обобщенных типов:
```csharp
public static void Main()
{
    Type t2 = Type.GetType("System.Collections.Generic.Dictionary`2");
    var mis2 = t2.GetMethods().Select(m => m.Name);
    foreach (var m in mis2)
        Console.WriteLine($"->{m}");
    Console.ReadLine();
}
```

Рефлексия параметров и возвращаемого значения метода
```csharp
public static void Main()
{
    Type t2 = typeof(int);
    MethodInfo[] mi = t2.GetMethods();
    foreach (var m in mi)
    {
        string returnVal = m.ReturnType.FullName;
        string paramInfo = "( ";
        foreach (var pi in m.GetParameters())
        {
            paramInfo += $"{pi.ParameterType} {pi.Name} ";
        }
        paramInfo += " )";
        Console.WriteLine($"->{returnVal} {m.Name} {paramInfo}");
    }
    Console.ReadLine();
}
```
То-эе самое:
```csharp
public static void Main()
{
    Type t2 = typeof(int);
    var methodName = t2.GetMethods().Select(n => n);
    foreach (var m in methodName)
        Console.WriteLine($"->{m}");
    Console.ReadLine();
}
```

## Динамическая загрузка сборок

Пример загрузки сборки находящейся рядом с испольняемым файлом и его рефлексия:
```csharp
public static void Main()
{
    string asmName = "ClassLibrary1"; //дружественное имя сборки
    try
    {
        Assembly asm = Assembly.Load(asmName);
        Console.WriteLine($"->{asm.FullName}");
        var types = asm.GetTypes();
        foreach (var t in types)
            Console.WriteLine($"Type: {t}");
    }
    catch
    {
        Console.WriteLine("Сборка не найдена");
    }
    Console.ReadLine();
}
```
Есть возможность загружать сборку по абсолютному путю:
```csharp
public static void Main()
{
    string asmName = "ClassLibrary1.dll"; //абсолютный путь
    try
    {
        Assembly asm = Assembly.LoadFrom(asmName);
        Console.WriteLine($"->{asm.FullName}");
        var types = asm.GetTypes();
        foreach (var t in types)
            Console.WriteLine($"Type: {t}");
    }
    catch
    {
        Console.WriteLine("Сборка не найдена");
    }
    Console.ReadLine();
}
```
Пример загрузки разделяемой сборки и его рефлексия:
```csharp
public static void Main()
{
    try
    {
        string asmStr = @"ClassLibrary1,Version=1.0.0.0,PublicKeyToken=2ebffce843c773fc,Culture=""";
        Assembly a = Assembly.Load(asmStr);
        Console.WriteLine($"->{a.FullName}");
        var types = a.GetTypes();
        foreach (var t in types)
            Console.WriteLine($"Type: {t}");
    }
    catch (Exception ex)
    {
        Console.WriteLine("Сборка не найдена" + ex.Message);
    }
    Console.ReadLine();
}
```
Загрузка этой же сборки путем ООП:
```csharp
AssemblyName asmName = new AssemblyName();
asmName.Name = "ClassLibrary1";
asmName.Version = new Version("1.0.0.0");
asmName.SetPublicKeyToken(new byte[] {0x2e,0xbf,0xfc,0xe8,0x43,0xc7,0x73,0xfc});
asmName.CultureName = "";
Assembly a = Assembly.Load(asmName);
```

## Позднее связывание

Осуществляется использованием класса System.Activator и метода CreateInstance().

Пример:

```csharp
public static void Main()
{
    Assembly a = null;
    try
    {
        a = Assembly.Load("ClassLibrary1");
    }
    catch (FileNotFoundException e)
    {
        Console.WriteLine(e.Message); Console.ReadLine();
        return;
    }
    if (a != null)
    {
        try
        {
            var myClass = a.GetType("ClassLibrary1.Class1");
            var obj = Activator.CreateInstance(myClass);
            Console.WriteLine($"Создано {obj} поздним связыванием.");
        }
        catch (Exception e)
        {
            Console.WriteLine(e.Message); Console.ReadLine();
        }
    }
    Console.ReadLine();
}
```
Вызов методов без параметров:
```csharp
public static void Main()
{
...
    if (a != null)
    {
        try
        {
            var myClass = a.GetType("ClassLibrary1.Class1");
            var obj = Activator.CreateInstance(myClass); //получение класса
            var mi = myClass.GetMethod("GetSalary"); //получение метода
            mi.Invoke(obj, null); //вызов
        }
        catch (Exception e)
        {
            Console.WriteLine(e.Message); Console.ReadLine();
            return;
        }
    }
    Console.ReadLine();
}
```
Вызов методов с параметрами:
```csharp
public static void Main()
{
...
    if (a != null)
    {
        try
        {
            var myClass = a.GetType("ClassLibrary1.Class1");
            var obj = Activator.CreateInstance(myClass);
            var mi = myClass.GetMethod("SetAge");
            mi.Invoke(obj, new object[]{19});
        }
        catch (Exception e)
        {
            Console.WriteLine(e.Message); Console.ReadLine();
            return;
        }
    }
    Console.ReadLine();
}
```

## Атрибуты

Некоторые предопределенные атрибуты:
```csharp
[CLSCompliant]      Элемент будет соответствовать правилам CLS
[DllImport]         Обращение кода .NET к любой неуправляемой библиотеке кода С или С++
[Obsolete]          Помечает устаревший тип или член. Если другие прогеры попытаются использовать его, они получат предупреждение
[Serializable]      Класс или структура будут помечены как сериализуемые
[NonSerialized]     Указание что данное поле в классе или структ. не должно сохранятся в процессе сериализации
[ServiceContract]   Пометка метода как контракт службы WCF
```
Примеры применения:
```csharp
[Serializable, Obsolete("Старый класс!")]
public class Class1
{
...
}
[Serializable]
[Obsolete("Старый класс!")]
public class Class1
{
...
}
```
## Специальные атрибуты

Пример сконструированного атрибута:
```csharp
public sealed class MyAttribute : Attribute //должен бытьь завершенным - sealed
{
    public string Description { get; set; }
    public MyAttribute(string description)
        => Description = description;
    public MyAttribute() {}
}
```
Пример применения специальных атрибутов:
```csharp
[MyNew("текст для примера")]
public class MyClass
{
    [MyNew("Новый")]
    public void Go()
    {}
}
```
Пример наложения ограничения на применение сконструированного атрибута:

## Атрибуты сборки
```csharp
[assembly:CLSCompliant(true)]
namespace ConsoleApp1
{
...
```

## Рефлексия атрибутов
Пример с ранним связыванием:
```csharp
Type t = typeof(MyClass);
var customAtts = t.GetCustomAttributes(false);
foreach (var c in customAtts)
    Console.WriteLine($"-> {(c as MyNewAttribute)?.Description}"); //вывод описания специального аттрибута
```

Пример с поздним связыванием:
```csharp
public static void Main()
{
    try
    {
        Assembly asm = Assembly.Load("ClassLibrary1");
        Type myDesc = asm.GetType("ClassLibrary1.MyNewAttribute");
        PropertyInfo prop1 = myDesc.GetProperty("Description");
        var types = asm.GetTypes();
        foreach (var t in types)
        {
            var objs = t.GetCustomAttributes(myDesc, false);
            foreach (var o in objs)
            {
                Console.WriteLine($"-> {t.Name}: {prop1.GetValue(o, null)}");
            }
        }
    }
    catch (Exception e)
    {
        Console.WriteLine(e.Message);
    }
    Console.ReadLine();
}
```

## Расширяемое приложение

Сборка определения типов, которые будут использоватся каждым объектом оснастки, на нее будет ссылаться расширяемое приложение:

Файл "MySnappableTypes.dll":
```csharp
namespace MySnappableTypes
{
    public interface IAppFunctionality //интерфейс для всех остасток, которые могут потребляться расширяемым приложением
    {
        void DoIt();
    }
    [AttributeUsage(AttributeTargets.Class)]
    public sealed class MyInfoAttribute : Attribute //спец атрибут, который может применятся к любому классу, желающему подключится к контейнеру
    {
        public string CompanyName { get; set; }
    }
}
```
Оснастка, в которой задействованы типы из сборки выше (подключить ссылку на сборку выше):

Файл "MySnapIn.dll":
```csharp
using MySnappableTypes;
namespace MySnapIn
{
    [MyInfo(CompanyName = "MyTest")]
    class MyModule : IAppFunctionality
    {
        public void DoIt()
        {
            Console.WriteLine("Успешное применение оснастки 'MyTest'!");
        }
    }
}
```
Расширяемое консольное приложение, которое может быть расширено функциональностью каждой оснастки.

Приложение (подключить ссылку на предидущую сборку):
```csharp
using MySnappableTypes;
namespace ConsoleApp1
{
    class Program
    {
        [STAThread]
        public static void Main()
        {
            try
            {
                LoadSnapIn(); //загрузка оснастки
            }
            catch (Exception e)
            {
                Console.WriteLine(e.Message);
            }
            Console.ReadLine();
        }
        static void LoadSnapIn()
        {
            OpenFileDialog dlg = new OpenFileDialog
            {
                InitialDirectory = Path.GetDirectoryName(Assembly.GetExecutingAssembly().Location),
                Filter = "Assemblies (*.dll)|*.dll|All files (*.*)|*.*",
                FilterIndex = 1,
            };
            if (dlg.ShowDialog() == DialogResult.OK)
            {
                if (dlg.FileName.Contains("MySnappableTypes"))
                    Console.WriteLine("В библиотеке нет оснасток!");
                else if (!LoadExternalModule(dlg.FileName))
                    Console.WriteLine("Не включает интерфейс IAppFunctionality");
            }
        }
        //загрузка оснастки и ее тестирование
        static bool LoadExternalModule(string path)
        {
            bool foundSnapIn = false;
            Assembly asm = null;
            try
            {
                asm = Assembly.LoadFrom(path);
            }
            catch (Exception e)
            {
                Console.WriteLine($"Ошибка загрузки оснастки: {e.Message}");
                return foundSnapIn;
            }
            var theClassTypes = asm.GetTypes()
                .Where(t => t.IsClass && t.GetInterface("IAppFunctionality") != null)
                .Select(t => t); //получить все классы совместимые с интерфейсом IAppFunctionality
            foreach (var t in theClassTypes)
            {
                foundSnapIn = true;
                var itfApp = asm.CreateInstance(t.FullName, true) as IAppFunctionality;
                itfApp?.DoIt(); //выполнить метод из оснастки
                DisplaySnapInfo(t); //отобразить информацию
            }
            return foundSnapIn;
        }
        //отображение информацию метаданных
        private static void DisplaySnapInfo(Type tp)
        {
            var compInfo = tp.GetCustomAttributes(false).Where(t => t is MyInfoAttribute).Select(t => t); //получить данные
            foreach (var c in compInfo)
            {
                Console.WriteLine($"Информация о сборке: {(c as MyInfoAttribute)?.CompanyName}"); //отобразить данные
            }
        }
    }
}
```
