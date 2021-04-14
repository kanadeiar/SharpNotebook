# Консоль

## Простая программа:
```csharp
using System;
namespace Lesson1
{
  class Program
  {
      static void Main()
      {
      }
  }
}
```
Варианты метода Main() (точка входа приложения):
```csharp
static int Main(string [ ] args)
{
    return 0;
}
static void Main()
{
}
static int Main ()
{
    return 0;
}
```
По соглашению возврат значения 0 указывает на то, что программа завершилась успешно, тогда как любое другое значение (вроде -1) представляет условие ошибки (имейте в виду, что значение 0 автоматически возвращается даже в случае, если метод Мат () прототипирован для возвращения void).

## Обработка аргументов командной строки
```csharp
foreach(string arg in args)
    Console.WriteLine("Arg: {0}", arg);
```
либо, если допустим применен метод Main без параметров:
```csharp
string[] theArgs = Environment.GetCommandLineArgs();
foreach(string arg in theArgs)
    Console.WriteLine("Arg: {0}", arg);
```
вызов приложения с аргументами: SimpleCSharpApp.ехе /argl -arg2

## Команды вывода результата в окошко консоли:
Статический метод WriteLine () помещает в поток вывода строку текста (включая символ возврата каретки).
Метод Write() помещает в поток вывода текст без символа возврата каретки.
```csharp
Console.Write("Нет перехода");
Console.WriteLine("Переход на след строку");
```
>Ввести cw два раза Tab - сокращение Console.WriteLine().

## Управляющие последовательности символов:
Управляющая последовательность      Описание

\a                                  Звуковой сигнал, аудио - подсказка пользователю

\n                                  Новая строка (перевод строки) 

\r                                  Возврат каретки

\t                                  Горизонтальная табуляция

\'                                  Одинарная кавычка

\"                                  Двойная кавычка

\\                                  Обратная косая черта

## Форматированный вывод чисел:
```csharp
Console.WriteLine ("{1:d}, {0:d2}, {2:d9}", 10, 20, 30);
```
Тип форматирования          Код формата         Результат

Decimal (десятичный)        D                   12345

D1                  12345
                            
D7                  0012345
                            
Fixed point (с фикс точкой) F                   12345.00

F1                  12345.0
                            
F7                  12345.0000000

```csharp
Console.WriteLine("c format: {0:c}", 99999); //рубли 99 999,00 ?
Console.WriteLine("d9 format: {0:d9}", 99999); //десятичные числа 000099999
Console.WriteLine("f3 format: {0:f3}", 99999); //с плавающей точкой 99999,000
Console.WriteLine("n format: {0:n}", 99999); //понятные с плавающей точкой 99 999,00
Console.WriteLine ("Е format: {0:Е}", 99999); //с экпонентой 9,999900e+004
Console.WriteLine ("е format: {0:е}", 99999);
Console.WriteLine("X format: {0:X}", 99999); //шестрандацатеричные 1869f
Console.WriteLine("x format: {0:x}", 99999);
```

## Команда ввода данных с консольного окошка
Метод ReadLine () позволяет получить информацию из потока ввода вплоть до нажатия клавиши <Enter>.
Метод Read() используется для захвата одиночного символа из потока ввода.
```csharp
string str = Console.ReadLine();
```
    
## Интерполирование строк
Указывается знак доллара - команда осуществления интерполяция строк.
```csharp
Console.WriteLine($"Имя: {person.Name}  Возраст: {person.Age}");
string result = $"{x} + {y} = {x + y}";
string output = $"{person?.Name??"Имя по умолчанию"}";
Console.WriteLine($"{number:+# ### ### ## ##}");
Console.WriteLine($"{number:F3,20}");
Console.WriteLine($"Coin flip: {(rand.NextDouble() < 0.5 ? "heads" : "tails")}");
string message = $"{date,20}{number,20:N3}";
```


    
    

# Продвинутое

## Русификация консоли в .NET 5

Гарантированная русификация в новых версиях консолей:
```csharp
static void Main(string[] args)
{
    [DllImport("kernel32.dll")] static extern bool SetConsoleCP(uint pagenum);
    [DllImport("kernel32.dll")] static extern bool SetConsoleOutputCP(uint pagenum);
    SetConsoleCP(65001);        //установка кодовой страницы utf-8 (Unicode) для вводного потока
    SetConsoleOutputCP(65001);  //установка кодовой страницы utf-8 (Unicode) для выводного потока
    Console.WriteLine("Приложение завершило свою работу");
    Console.ReadKey();
}
```

## Асинхронные методы main()
Используются при асинхронном программировании.
```csharp
static Task Main ()
static Task<int> Main()
static Task Main(string[])
static Task<int> Main(string[])
```

## Дополнительные члены класса System.Enviroment
Показ информации о среде выполения:
```csharp
static void ShowEnvironmentDetails ()
{
    // Вывести информацию о дисковых устройствах
    // данной машины и другие интересные детали,
    foreach (string drive in Environment.GetLogicalDrives())
        Console.WriteLine("Drive: {0}", drive); // Логические устройства
    Console.WriteLine("OS: {0}", Environment.OSVersion) ; // Версия
    // операционной системы
    Console.WriteLine("Number of processors: {0}", Environment.ProcessorCount); // Количество процессоров
    Console.WriteLine (".NET Version: {0}", Environment.Version); // Версия платформы .NET
```
## Интересные члены класса System.Console

Beep() - звуковой сигнал определенной частоны, длительности, например Beep(500,500)

BackgroundColor() - цвет фона

Title - заголовок консоли

BufferHeight BufferWidth размер буферной области консоли

Clear() - очистить вывод консоли

WindowHeight WindowWidth WindowTop WindowLeft - размер окна



