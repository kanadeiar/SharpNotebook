# Динамические типы и DLR (позднее связывание, COM, Excel, DBF, Simatic)

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
Пример использования того-же самого:
```csharp
using Excel = Microsoft.Office.Interop.Excel;
var excel = new Excel.Application();
var workbook = excel.Workbooks.Add();
workbook.Sheets.Add();
var worksheet = workbook.Sheets[numSheet];
worksheet.Name = sheetName;
worksheet.Columns[1].ColumnWidth = 4; //ширина колонок
worksheet.Columns[2].ColumnWidth = 16;
worksheet.Cells[1, 1] = "№";
worksheet.Cells[1, i + 6].Formula = $"=SUM(F2:F6)"; //формула
worksheet.Range[$"A1:Z{list.Count + 1}"].AutoFormat(Excel.XlRangeAutoFormat.xlRangeAutoFormatClassic1); //форматирование листа
```
Пример использования COM-библиотеки в приложении для открытия файла Excel и быстрой записи большого объема информации в него:
```csharp
using Excel = Microsoft.Office.Interop.Excel;
...
string[] strs =
{
    "25.05.2020 0:00:55;42,9799995422363;25.05.2020 0:00:55;44,4000015258789;25.05.2020 0:00:55;54;25.05.2020 0:00:55;527;25.05.2020 0:00:55;82,2799987792969",
    "25.05.2020 0:01:55; 43,1800003051758; 25.05.2020 0:01:55; 44,4000015258789; 25.05.2020 0:01:55; 54; 25.05.2020 0:01:55; 528; 25.05.2020 0:01:55; 82,879997253418",
    "25.05.2020 0:00:55;42,9799995422363;25.05.2020 0:00:55;44,4000015258789;25.05.2020 0:00:55;54;25.05.2020 0:00:55;527;25.05.2020 0:00:55;82,2799987792969",
};
Excel.Application application = new Excel.Application();
var workbook = application.Workbooks.Open("excel.xslx", 0, true, 5, "", "", false);
Excel.Worksheet worksheet = (Excel.Worksheet)workbook.Sheets[1];
int currLine = 0;

int rowCount = strs.Length;
int colCount = 6;
object[,] cells = new object[rowCount, colCount];
foreach (var el in strs)
{
    var split = el.Split(';');
    string Date = split[0].Remove(split[0].LastIndexOf(':'));
    string Power = split[1];
    string Frenq = split[3];
    string Temp = split[5];
    string Voltage = split[7];
    string Currency = split[9];
    cells[currLine, 0] = Date;
    cells[currLine, 1] = Power.Replace(',', '.');
    cells[currLine, 2] = Frenq.Replace(',', '.');
    cells[currLine, 3] = Temp.Replace(',', '.');
    cells[currLine, 4] = Voltage.Replace(',', '.');
    cells[currLine, 5] = Currency.Replace(',', '.');
    currLine++;
}            
worksheet.get_Range((Microsoft.Office.Interop.Excel.Range)(worksheet.Cells[2, 1]), (Microsoft.Office.Interop.Excel.Range)(worksheet.Cells[rowCount + 1 + 5, colCount])).Value = cells;  //быстрая запись массива ячеек в лист эксель
((Excel.Worksheet)workbook.Sheets[2]).Activate();
application.Visible = true;
application.UserControl = true;
//workbook.Save();
//workbook.Close();
```
Пример использования COM-библиотеки в приложении для формирования письма Outlook:
```csharp
using COutlook = Microsoft.Office.Interop.Outlook;
...
AppSettingsReader ar = new AppSettingsReader();
COutlook.Application app = new COutlook.Application();
COutlook.MailItem mailItem = app.CreateItem(COutlook.OlItemType.olMailItem);
mailItem.Subject = "Тема";
mailItem.To = "exsample@ex.ru";
mailItem.Body = "Просто тестовое письмо";
if (attachments != null)
{
    foreach (var a in attachments)
    {
        mailItem.Attachments.Add(a);
    }
}
mailItem.Importance = COutlook.OlImportance.olImportanceNormal;
mailItem.Display(false);
```

## Взаимодействие с DBF

Пакет: NDbfReaderEx

```csharp
using (DbfTable table = DbfTable.Open(FileDbf1, Encoding.GetEncoding(437)))
{
    var row = table.GetRow(i + 3);
    var Number = Int32.Parse(row.GetString("0001")),
}
```

## Взаимодействие с Simatic

Пакет: Sharp7

```csharp
using Sharp7;
static S7Client s7client = new S7Client();
rez = s7client.ReadArea(S7Consts.S7AreaMK, 0, 282, 4, S7Consts.S7WLByte, buffer); //меркерты, 282 смещение в памяти, длинна - 4 байта
fromMixToVT1 = (buffer[1] & 0b100000) != 0;
fromMixToVT1Work = (buffer[2] & 0b10) != 0;
s7client.DBRead(201, 100, 2, buffer); //блок данных 201, смещение в нем 100, длинна - 2 байта
valuePath = S7.GetIntAt(buffer, 0);
```
