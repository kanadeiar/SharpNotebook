# Коллекции

## Необобщенные коллекции
Для работы с этими коллекциями требуется подключить пространство имен System.Collections;

Представлены такими классами, как ArrayList
```csharp
ArrayList list = new ArrayList();
list.Add("1");
list.Sort();
foreach (var v in list) Console.WriteLine(v);
```
Такие коллекции не следует применять в приложениях, так как состоят из ссылки на объекты общего для всех типа object и за списком приходится следить программисту.

## Обобщенные коллекции
Для работы с этими коллекциями требуется подключить пространство имен System.Collections.Generic. 

Представлены такими классами, как List<>
```csharp
List<string> list = new List<string>();
list.Add("1");
list.Sort();
foreach (var v in list) Console.WriteLine(v);
```



