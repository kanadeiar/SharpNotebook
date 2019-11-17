# Функции

Функции и методы - технически это одно и тоже, только методы принадлежат классу, а функции - нет. Метод описывается внутри класса. Что какое класс написано в другом разделе, не здесь. Методы можно объявлять статическими, тогда он принадлежит классу, а для его вызова не нужно создавать объект класса.

То есть просто статический метод и есть "функция".

```csharp
модификаторы  тип вертаемого   имя     параметры
    public static void MyFunc(str string)
    {
        ...         тело метода
    }  
```

## Вызов метода.
Круглые скобки после названия - это значит метод.
```csharp
class My Class
{
    static void MyFunc()
    {
        Console.ReadKey();
    }
    static void Main()
    {
        MyFunc();
    }    
}
```
## Вертаемое значение функции (метода).
Вместо void поставить тип - тогда через return можно вертать значения.
```csharp
    static int MyFunc()
    {
        return 1;
    }
```
Вызов:
```csharp
    int i = MyFunc();
```
## Перегрузка методов.
Методы с одинаковыми именами, но разными параметраами являются перегруженными методами.
```csharp
static void Pause()                    
{
    Console.ReadKey();
}
static void Pause()   //это перегруженный метод
{
    Console.WriteLine("fdddfffggffg");
    Console.ReadKey();
}
```
## Модификаторы параметров методов
Передача параметра в метод по ссылке - ref
```csharp
static void PlusPlus(ref int a)
{
    a++;
}
```
Передача по ссылке параметра без опязательной первоначальной инициализации - out
```csharp
static void PlusPlus(out int a)
{
    a = 1;
}
```

## Процедурное программирование.
Программирование, при котором все выполняемые операторы собираются в подпрограммы, в более крупные еденицы кода.
Исходный код:
```csharp
static void Main()
{
    Console.WriteLine("Введите вещественное число a:");
    double a = double.Parse(ReadLine());
    a = Math.Pow(a, 3);
    Console.WriteLine($"a в степени 3 равено {a}");
    ReadKey();
}
```
Можно выделить операции ввода чисел, логической обработки и вывода.
```csharp
static double InputDouble()
{
    Console.WriteLine("Введите вещественное число a:");
    return double.Parse(ReadLine());
}
static double CalcPowThreee(double a)
{
    a = Math.Pow(a, 3);
}
static void WriteDoubleToConsole(double a)
{
    Console.WriteLine($"a в степени 3 равено {a}");
}
static void Main()
{
    double a = InputDouble();
    a = CalcPowThreee(a);
    WriteDoubleToConsole(a);
    ReadKey();
}
```
Взаимозаменяемые функции можно заменять другими. Умение представить программу в виде подпрограмм является очень важным умением программиста.



