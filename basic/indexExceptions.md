## Исключения

Конструкция обработки исключений try-catch-finally позволяет обрабатывать любые непридвиденне и ошибочные ситуации, которые могут произойти при выполнении кода.

Конструкция:
```csharp
try
{
    //код, в которых может возникнуть исключение
}
catch
{
    //код для обратки исключения
}
finally
{
    //выполняемый в любом случае код
}
```

Простой пример:
```csharp
try
{
    int a = Convert.ToInt32(Console.ReadLine());
}
catch (FormatException ex)
{
    Console.WriteLine(ex.Message);
}
catch (Exception ex)
{
    Console.WriteLine(ex.Message);
}
finally
{
    Console.WriteLine("End");
}
```



