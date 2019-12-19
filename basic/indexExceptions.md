## Исключения

Три аномалии в коде:

- дефекты - ошибки, которые допустил сам программист.

- пользовательские ошибки - возникает по причине неправильного использования пиложения пользователем, если не была предусмотрена защита от такого использования.

- исключения - аномалии которые трудно учесть на стадии программирования приложения - программист обладает низким контролем над такими исключительными обстоятельствами.

Исключения в .NET представляют собой объекты, в которых содержится читаблеьное описание проблемы, а также детальный снимок стека вызовов на момент первоначального возникновения исключения, и плюс еще маленькую тележку данных может сам программист прицепить сюда же.

Исключения состоят из:

- тип класса, который представляет детали исключения.

- член, способный генерировать экземплр класса исключения в вызывающем коде при определенных обстоятельствах.

- блок кода на вызывающей стороне, который обращается к члену, предрасположенному к возникновению исключений.

- блок кода на вызывающей стороне, который должен будет обрбатывать исключение, когда оно произойдет.

## Базовый класс исключений.

Все исключение происходят от базового класа System.Exception, который является System.Object. 

Основные члены этого класса:
```csharp
Data            Позволяет извлекать коллекцию пар "ключ-значение", представляющую доп инфу об исплючении, определенной программистом.
HelpLink        Свойство с URL для доступа к справочному файлу или сайту
InnerException  Инфа о предидущих исключениях, которые послужили причиной возникновения тек. исключения.
Message         Текстовое описание заданной ошибки.
Source          Имя сборки или объекта, который привел к генерации исключения.
StackTrace      Строка, инедтифицирующая последовательность вызовов, которая привела к возникновению исключения.
TargetSite      Объект с описанием многочисленных деталей о метода, который привел к генерации исключения.
```
Простейший способ генерации исключения:
```csharp
public class Sample
{
    public int Div(int one, int two)
    {
        if (two == 0)
            throw new Exception("На ноль делить нельзя!!!");
        return one / two;
    }
}
```
Простой пример перехвата исключений:
```csharp
Sample sample = new Sample();
try
{
    int rez = sample.Div(10, 0);
}
catch (Exception e)
{
    WriteLine(e.Message);
}
```
Если сработает исключение, то выполняется блок кода catch, если нет - блок catch пропускается.

## Свойства исключения

Свойство TargetSite позволяет выяснить делати о методе, который сгенерил исключение. Пример:
```csharp
WriteLine("Member name: " + e.TargetSite); //Int32 Div (Int32, Int32)
WriteLine("Member type: " + e.TargetSite.MemberType); //Method
WriteLine("Source: " + e.Source); //ConsoleApp1Test
```

Свойство StackTrace позволяет идентифицировать последовательность вызовов, которая в результате привела к генерации исключения. Пример:
```csharp
WriteLine("Stack: {0}", e.StackTrace); //Stack:    в ConsoleApp1Test.Sample.Div(Int32 one, Int32 two) в D:\Develop\gitSharp\LearnForBook\ConsoleApp1Test\Program.cs:строка 154
//в ConsoleApp1Test.Program.Main(String[] args) в D:\Develop\gitSharp\LearnForBook\ConsoleApp1Test\Program.cs:строка 191
```
>Отражает последовательность вызовов, которая привела к генераци данного исключения. Самый нижний номер строки - место возникновения первого вызова в последовательности, а самый верхний - место, где точно находится проблемный член.

Свойство HelpLink может быть полезным пользователю - специальный URL или файл, где приводятся подробные сведения о проблеме. Пример исключения:
```csharp
Exception ex = new Exception("На ноль делить нельзя!!!");
ex.HelpLink = "http://www.yandex.ru";
throw ex;
его обработка / перехват:
WriteLine("Help Link: {0}", e.HelpLink);
```

Свойство Data позволяет заполнить объект исключения подходящей вспомогательной информацией. Это свойство возвращает объект, который реализует интерфейс IDictionary. Пример:
```csharp
if (two == 0)
{
    Exception ex = new Exception("На ноль делить нельзя!!!");
    ex.Data.Add("TimeStamp", $"Время срабатывания {DateTime.Now}");
    ex.Data.Add("Lot", "Куча данных");
    throw ex;
}
//обработка:
WriteLine("Специальные данные:");
foreach (DictionaryEntry el in e.Data)
{
    WriteLine($"-> {el.Key} {el.Value}");
}
```

## Исключения уровня системы

Исключения, которые генерируются самой платформой .NET, называются системными исключениями, эти исключения рассматриваются как неисправимые фатальные ошибки. Они унаследованы от класса System.SystemException а он является System.Exception а он является System.Object.

Проверка что исключение - системное:
```csharp
NullReferenceException nullex = new NullReferenceException();
WriteLine($"Is SystemException -> {nullex is SystemException}");
```

## Исключения уровня приложения

Исключения, производные от класса System.ApplicationException, созданные разработчиком, будут иметь идентификацию, что они не системные, а уровня приложения. Можно предполагать, что такие исключения созданы кодовой базой приложения, а не библиотекой базовый классов или исполняющей средой.

Пример:
```csharp
ApplicationException ex = new ApplicationException("На ноль делить нельзя");
throw ex;
//обработка:
catch (ApplicationException e)
{
    WriteLine("111");
}
catch (Exception e)
{
    WriteLine(e.Message);
}
```

## Построение специальных исключений

Для построения своего типа исключения нужно прежде всего создать класс унаследованный от Exception или ApplicationException. 

Пример специального исключения:
```csharp
public class MyException : ApplicationException
{
    private string messDetails = string.Empty;
    public MyException() {}
    public MyException(string mess)
    {
        messDetails = mess;
    }
    public override string ToString() => $"My Exc {messDetails}";
}
//использование
if (two == 0)
{
    MyException ex = new MyException("НООООЛЬ!!!");
    throw ex;
}
//обработка
catch (MyException e)
{
    WriteLine(e.ToString());
}
```

>Все спец классы исключений должны быть публичными - они могут передаваться за границы сборок.

Другой более простой вариант построения спец исключения:
```csharp
public class MyException : ApplicationException
{
    public string MyDeatils { get; set; }
    public MyException(string message, string details) : base(message)
    {
        MyDeatils = details;
    }
}
//использование
MyException ex = new MyException("Деление на ноль!!!", "Пипец");
throw ex;
//обработка
catch (MyException e)
{
    WriteLine(e.Message);
    WriteLine(e.MyDeatils);
}
```
Тут используется концепция наследования - входное сообщение передается конструктору родительского класса.

Третий самый навороченный вариант исключения - это полное исключение .NET платформы со всеми определениями. Сделать его - ввести Ex и два раза таб.

Пример шаблона:
```csharp
[Serializable]
public class MyException : Exception
{
    public MyException()
    {
    }

    public MyException(string message) : base(message)
    {
    }

    public MyException(string message, Exception inner) : base(message, inner)
    {
    }

    protected MyException(
        SerializationInfo info,
        StreamingContext context) : base(info, context)
    {
    }
}
```





