# Динамические типы и DLR (позднее связывание, COM, Excel)

В .NET версии 4.0 появилось ключевое слово dynamic, позволяющее объявлять не строго типизированную переменную. Она не типизирована статически. Способна принимать идентичность любого типа на лету.

Важное отличие от System.Object в том, что динамические данные не являются статически типизированными. Компилятор допустимых членов проверять не будет!

Пример сравнения (результат у всех будет System.String):
```csharp
var s1 = "One";
object s2 = "Two";
dynamic s3 = "Three";
Console.WriteLine($"Тип: {s1.GetType()}");
Console.WriteLine($"Тип: {s2.GetType()}");
Console.WriteLine($"Тип: {s3.GetType()}");
```

Класс с динамическими членами:
```csharp
class MyClass
{
    private static dynamic myField;
    public dynamic DynProperty { get; set; }
    public dynamic DynMethod(dynamic dynParam)
    {
        dynamic dynLocVar = "Local variable";
        int myInt = 0;
        if (dynParam is int)
        {
            return dynLocVar;
        }
        return myInt;
    }
}
```

## Позднее связывание с применением динамических типов

Пример старая версия кода:
```csharp
static void MyUsingLateBinding(Assembly asm)
{
    try
    {
        Type myClass = asm.GetType("ClassLibrary1.Class1");
        object obj = Activator.CreateInstance(myClass);
        MethodInfo mi = myClass.GetMethod("SetAge");
        mi.Invoke(obj, null);
    }
    catch (Exception e)
    {
        Console.WriteLine(e.Message);
    }
}
```

Пример новая версия кода с применением динамических типов:
```csharp
static void MyUsingLateBinding(Assembly asm)
{
    try
    {
        Type myClass = asm.GetType("ClassLibrary1.Class1");
        dynamic obj = Activator.CreateInstance(myClass); //создать экземпляр на лету
        obj.SetAge();
    }
    catch (Exception e)
    {
        Console.WriteLine(e.Message);
    }
}
```

Пример передачи аргументов с применением динамических типов:
```csharp
//библиотека
namespace ClassLibrary1
{
    public class Class1
    {
        public int Add(int x, int y) => x + y;
    }
}
class Program
{
    public static void Main()
    {
        Assembly asm = Assembly.Load("ClassLibrary1");
        try
        {
            Type myClass1 = asm.GetType("ClassLibrary1.Class1");
            dynamic obj = Activator.CreateInstance(myClass1);
            Console.WriteLine($"Результат: {obj.Add(1,2)}");
        }
        catch (Exception e)
        {
            Console.WriteLine(e.Message);
        }
        Console.ReadLine();
    }
}
```

## Взаимодействие с COM с динамическими типами данных

Библиотеки COM содержат метаданные подобно .NET, но в другом формате. Для взаимодействия программы .NET с объектом COM, потребуется сгенерировать сборку взаимодействия.

Среда разработки при добавлении ссылки на COM библиотеку, сама генерирует новую сборку, которая включает описания .NET метаданных COM. За кулисами данные маршалирируются между приложениями .NET и COM. Многие библиотеки COM поставляемые средой разработки, предоставляют официальную сборку взаимодействия (PIA).

Сборки PIA перечислены в разделе "Сборки" диалогового окра в разделе Расширения. Пакет "Набор средств Visual Studio для Office (VSTO)" - установить. Добавить "Microsoft.Office.Interop.Excel". С выхода .NET 4.0 эти взаимодействия можно встраивать прямо в скомпилированное приложение. Для этого достаточно установить свойство EmbedInteropTypes в True. Этим сокращается размер приложения, поставляемого клиенту, так как не придется устанавливать сборки PIA, которые отсутствуют на целевой машине.

Многие методы COM проектировались так, чтоб принимать и возвращать специфический тип данных Variant, во многом похожий на dynamic. Когда свойство EmbedInteropTypes установлено, все COM-типы Variant отображаются на динамические данные. Это сильно упрощает работу с COM.

Пример использования COM-библиотеки в приложении для создания нового файла Excel:
```csharp
using Excel = Microsoft.Office.Interop.Excel;
namespace ConsoleApp1
{
    class Program
    {
        public static void Main()
        {
            Excel.Application excelApp = new Excel.Application();
            excelApp.Workbooks.Add();
            Excel.Worksheet worksheet = excelApp.ActiveSheet;
            worksheet.Cells[1, "A"] = "Fam";
            worksheet.Cells[2, "B"] = "Name";
            worksheet.Cells[3, "C"] = "Age";
            for (int i = 0; i <= 5; i++)
            {
                worksheet.Cells[i + 1, "A"] = $"Фамилия{i}";
                worksheet.Cells[i + 1, "B"] = $"{i}-е имя";
                worksheet.Cells[i + 1, "C"] = 18 + i;
            }           worksheet.Range["A1"].AutoFormat(Excel.XlRangeAutoFormat.xlRangeAutoFormatClassic2);
            excelApp.Visible = true;
            worksheet.SaveAs($@"{Environment.CurrentDirectory}\Test.xlsx");
            //excelApp.Quit();
            Console.ReadLine();
        }
    }
}
```
Пример использования COM-библиотеки в приложении для открытия файла Excel и записи в него:
```csharp
using Excel = Microsoft.Office.Interop.Excel;
namespace ConsoleApp1
{
    class Program
    {
        public static void Main()
        {
            Excel.Application application = new Excel.Application();
            var workbook = application.Workbooks.Open(toPath, 0, true, 5, "", "", false);
            Excel.Worksheet worksheet = (Excel.Worksheet)workbook.Sheets[1];
            string Date = DateTime.Now.ToString();
            string Power = "11,21";
            worksheet.Cells[currentLine, 1] = Date;
            worksheet.Cells[currentLine, 2] = Power.Replace(',','.');
            application.Visible = true;
            application.UserControl = true;
            //workbook.Save();
            //workbook.Close();
            Console.ReadLine();
        }
    }
}
```
