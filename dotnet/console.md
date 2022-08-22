# Консоль

## Варианты точек входа:

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
//асинхронные версии:
static Task Main()
static Task<int> Main()
static Task Main(string[])
static Task<int> Main(string[])
```

Начиная с c#9.0 можно использовать определения верхнего уровня, приложение может выглядеть так:

```csharp
using System;
Console.WriteLile("Sample");

Console.ReadKey();
```

Операторы верхнего уровня в версии C# 9.0 являются неявно асинхронными.

>Условия: операторы верхнего уровня можно использовать только в одном файле внутри приложения; программа не может иметь объявленную точку входа; нельзя помещать в пространство имен; возвращают код завершения приложения с использованием return; функции, становятся локальными функциями; дополнительные типы можно объявлять после всех операторов верхнего уровня. “За кулисами” компилятор заполняет пробелы

Обработка аргументов командной строки:

```csharp
// Обработка входных аргументов
for (int i = 0; i < args.Length; i++)
{
  Console.WriteLine("Arg: {0}", args[i]);
}
```

или так:

```csharp
string[] theArgs = Environment.GetCommandLineArgs();
foreach(string arg in theArgs)
    Console.WriteLine("Arg: {0}", arg);
```

Пример вызова приложения с аргументами: SimpleCSharpApp.ехе /argl -arg2

## Интересные члены класса System.Console

Beep() - звуковой сигнал определенной частоны, длительности, например Beep(500,500)

BackgroundColor() - цвет фона

Title - заголовок консоли

BufferHeight BufferWidth размер буферной области консоли

Clear() - очистить вывод консоли

WindowHeight WindowWidth WindowTop WindowLeft - размер окна

## Форматированный вывод чисел:

```csharp
Console.WriteLine ("{1:d}, {0:d2}, {2:d9}", 10, 20, 30);
```

Тип форматирования | Код формата | Результат
--------|--------|----------
Decimal (десятичный)     | D | 12345
| D1         |         12345                            
| D7         |         0012345                            
Fixed point (с фикс точкой) | F | 12345.00
| F1         |         12345.0                            
| F7         |        12345.0000000

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

## Управляющие последовательности символов:

Управляющая последовательность     | Описание
-------------------------|-------------------------
\a | Звуковой сигнал, аудио - подсказка пользователю
\n | Новая строка (перевод строки)
\r | Возврат каретки
\t | Горизонтальная табуляция
\' | Одинарная кавычка
\" | Двойная кавычка
\\ | Обратная косая черта


