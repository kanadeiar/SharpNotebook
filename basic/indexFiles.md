# Файлы

## Работа с текстовыми файлами
Для работы с текстовыми файлами в среде .NET Framework используется класс StreamReader из пространства имен System.IO.

Пример:
```csharp
StreamReader sr = new StreamReader("..\\..\\data.txt");
int n = int.Parse(sr.ReadLine());
for (int i = 0; i < n; i++)
{
    int a = int.Parse(sr.ReadLine());
    Console.WriteLine(a);
}
sr.Close();
```
