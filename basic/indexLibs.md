# Библиотека классов

## Класс математических операций
```csharp
Math.Pow(a,b); //возаращает а в степери b
```

## Класс получения расположения исполняемой сборки приложения
```csharp
string initDir = Path.GetDirectoryName(Assembly.GetExecutingAssembly().Location);
Console.WriteLine(initDir);
```

## Класс работы с массивом 
Array
```csharp
Имя метода      Тип             Описание
Clear           Статический     Присваивает элементам массива значение 0,
                                false или null в зависимости от типа элементов
GetLength       Нестатический   Получает целое число, представляющее 
                                количество элементов в заданном измерении              
Sort            Статический,    Сортирует элементы во всем одномерном массиве Array 
                                перегруженный                           
```

## Класс случайных значений
```csharp
Random rnd = new Random();
int variable = rnd.Next(0, 100);
```

## Получение прошедшего времени
```csharp
DateTime start=DateTime.Now;
System.Threading.Thread.Sleep(20);// делаем паузу
DateTime finish=DateTime.Now;
Console.WriteLine(finish-start);
```

## Замер скорости выполения кода:
```csharp
Stopwatch stopwatch = new Stopwatch();
stopwatch.Start();
/////////код для замера
stopwatch.Stop();
var elapsedTime = stopwatch.Elapsed; //результат работы
```

## Цикличная задержка программы в асинхронном режиме
```csharp
public static void Main()
{
    Run();
    Console.ReadLine();
    async void Run()
    {
        while (true)
        {
            Console.WriteLine("+");
            await Task.Delay(1000);
        }
    }
}
```

## Конфигурационный XML файл сборки.
Допустимо использовать набор типов из пространства имен System.Configuration для специальных настроек приложения.

Файл "App.config":
```csharp
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
...
    <appSettings>
      <add key="Str1" value="Green"/>
      <add key="Number1" value="8"/>
    </appSettings>
</configuration>
```
Использование:
```csharp
using System.Configuration;
namespace ConsoleApp1
{
    class Program
    {
        public static void Main()
        {
            AppSettingsReader ar = new AppSettingsReader();
            int numb1 = (int)ar.GetValue("Number1", typeof(int));
            string str1 = (string) ar.GetValue("Str1", typeof(string));
            Console.WriteLine($"Проверка: {str1} {numb1}");
```
