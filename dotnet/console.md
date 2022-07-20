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