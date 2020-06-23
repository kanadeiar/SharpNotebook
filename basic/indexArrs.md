# Массивы (индексаторы)

В языке C# массивы это набор однотипных элементов, расположенных рядом, объединенных одним общим имененм. Они аналогичны тем, что есть в других языках программирования. Важный момент что они относятся к ссылочным типам данных и реализованы как объекты. Имя массива - ссылка на объект в куче, в которой и размещается набор элементов определенного типа. 

Выделение памяти под элементв происходит на этапе инициализации массива. А вот за освобождение памяти можно не беспокоится - об этом позаботится автоматический сборщик мусора. Неиспользуемые массивы, как и объекты автоматически удаляются сборщиком мусора.

Если массив объявлен, но его элементы явно не заполнены по каждому индексу, то они получат стандартное значение для соответствующего типа данных (int - 0, bool - false).

## Одномерные массивы

Фиксированное количество элементов одного типа, под одним именем, при этом каждый элемент в этом массиве имеет свой номер. В языке C# нумерация массивов начинается с ноля. Массив является объектом, и поэтому как всякий объект создается в два этапа - объявляется ссылочная переменная, а затем выделяется память под требуемое количество элементов в куче. Тип массива определяет тип каждого элемента в массиве. Количество элементов, которые будут хранится в массиве задается его размером.

Можно применять синтаксис инициализации массивов - это указание всех элементов массива в фигурных скобках.

Можно применять неявно типизированные массивы используя ключевое слово var.

Примеры объявления одномерных массивов в C#:
```csharp
int[] array0;
array0=new int[5];
int[] array1 = new int[5];
int[] array2 = new int[] { 1, 3, 5, 7, 9 }; 
int[] array3 = { 1, 2, 3, 4, 5, 6 };
var array4 = new int[5];
var array5 = new int[] { 1, 3, 5, 7, 9 };
var array6 = new[] {1, 2, 3, 4, 5};
ver array7 = new[] {"hello", null, "World"}; //string
```
Обращение к элементам массивам происходит через квадратные скобки:
```csharp
int a = array[1];
array[2] = 3;
```
Массив может передаватся в метод как парметр, может быть результатом выполнения метода
```csharp
static int[] getArr(int[] b)
{
    return new[] {1, b[0], 5};
}
```
Массивы есть объекты:
```csharp
int a = array.Length;
```
Массив может быть из объектов, который может содержать и мух и котлеты рядом:
```csharp
objects[] myObjects = new objects[4];
```

## Многомерные (прямоугольные) массивы
Прямоугольные то что имеет несколько измерений, а содержащиемя в нем строки обладают одной и той же длинной.

Пример двумерного (и треножного) массива:
```csharp
int[,] multuArr1 = new int[2,2];
int[,] multiArr2 = { { 1, 2}, { 4, 5} };
int[,,] multiArr1 = {{{1,2}, {1,2}}, {{1,2}, {1,2}}};
WriteLine($"{multiArr2.Length}"); // 4
```
Для работы с двумерными массивами чаще всего используются вложенные циклы.

Также можно создавать ступенчатые массивы, или массивы массивов. Это одномерный массив, каждый элемент которого - еще один массив.
Пример ступенчатого массива:
```csharp
int[][] stepArray = new int[2][];
stepArray[0] = new []{1, 2, 4};
stepArray[1] = new []{1, 2, 4};
```
Пример работы с этим массивом:
```csharp
for(inti=0;i<myJagArray.Length;1++)
    myJagArray[i] = newint[i+7];
for(inti=0;i<5;1++)
    for(int3=0;j<myJagArray[1].Length;3++)
        Console.Write(myJagArray[1][j]+" ");
    Console.WriteLine()
```

## Индексаторы массивов
У массивов есть специальные свойства - индексаторы. Такие свойства позволяют получить публичный доступ к приватному полу класса - массиву. Это либо публичный метод, либо индексируемое свойство. В языке C# предлагается возможность проектирования специальных классов и структур, которые могут индексироваться подобно стандартному массиву, за счет определения индексаторного метода.

>Это наиболее полезно при создании специальных классов коллекций (обобщенных и необобщенных).

Синтаксис индексаторного свойства с применением лямда-выражений:
```csharp
public MyClass this[int index]
{
    get => arr[index];
    set => arr[index] = value;
}
```

Пример класса:
```csharp
class MySuperArray
{
    private int[] arr; //приватный массив
    public MySuperArray(int size) //конструктор
    {
        arr = new int[size];
    }
    public int Get(int i) //метод чтение элемента
    {
        return arr[i];
    }
    public void Set(int i, int value) //метод задание элемента
    {
        arr[i] = value;
    }
    public int this[int i] //индексируемое свойство - лучше всего рабоать этим
    {
        get { return arr[i];}
        set { arr[i] = value; }
    }
    public int this[int i, int plus] //индексируемое свойство с доп параметром
    {
        get { return arr[i] + plus; }
        set { arr[i] = value + plus; }
    }
    public ref int GetRef(int i) //через ссылку
    {
        return ref arr[i];
    }
}
```
Обращение к закрытому массиву этого класса различными способами:
```csharp
MySuperArray arr = new MySuperArray(10);
for (int i = 0; i < 10; i++)
    arr.Set(i, i); //через свойство
for (int i = 0; i < 10; i++)
    arr[i] = i; //через индексатор - лучше всего работать этим
for (int i = 0; i < 10; i++)
    arr.GetRef(i) = 10; //через ссылку
```

>Индексаторы являются еще одной формой "синтаксического сахара", учитывая, что ту же самую функциональность можно получить и с помощью старых открытых методов.

## Новый функционал работы с массивами

Пример:
```csharp
var arr = Array.CreateInstance(typeof(int), 10);
for (int i = 0; i < arr.Length; i++)
{
    arr.SetValue(arr.Length - i, i);
}
int elem = (int)arr.GetValue(0);
foreach (var el in arr)
    Console.Write($"{el} ");
Console.WriteLine();
```




# Продвинутое

## Использование строковых значений в индексаторном свойстве
Пример класса с индексаторным свойством - строкой:
```csharp
public class MyClass
{
    public string Name { get; set; }
    public string SurName { get; set; }
    public override string ToString()
    {
        return $"{SurName} {Name}";
    }
}
public class MyClassCollection : IEnumerable
{
    private Dictionary<string, MyClass> list = new Dictionary<string, MyClass>();

    public MyClass this[string name]
    {
        get => list[name];
        set => list[name] = value;
    }
    public void Clear()
    {
        list.Clear();
    }
    public int Count => list.Count;
    public IEnumerator GetEnumerator() => list.GetEnumerator();
}
```
Пример использования:
```csharp
MyClassCollection myCollect = new MyClassCollection();
myCollect["Max"] = new MyClass {Name = "Max", SurName = "Vik"};
myCollect["And"] = new MyClass {Name = "And", SurName = "Rot"};
WriteLine($"SurName = {myCollect["And"]}");
```
>Методы индексаторов могут быть перегружены в отдельном классе или структуре - по порядковой позиции, по дружественному строковому имени и по строковому имени с дополнительным пространством имен.

## Многомерные индексаторы
Пример класса с индексаторным свойством для доступа к многомерному массиву:
```csharp
public class DoubleCollection
{
    private int[,] myDou = new int[10, 10];
    public int this[int row, int col]
    {
        get => myDou[row, col];
        set => myDou[row, col] = value;
    }
}
```

## Индексаторы в интерфейсных типах
Пример использования интерфейса:
```csharp
public interface IStringContainer
{
    string this[int index] { get; set; }
}
public class MyClassInterfaceCollection : IStringContainer
{
    private List<string> list = new List<string>();
    public string this[int index]
    {
        get => list[index];
        set => list[index] = value;
    }
}
```

## Члены класса System.Array

Базовый массив получает массу функциональности от класса System.Array. Вот наиболее интересное что можно сделать с массивами:
```csharp
Clear() Этот статический метод устанавливает для заданного диапазона элементов в массиве пустые занчения (0 для чисел, null для объектных ссылок и false для булевских значений)
CopyTo() Этот метод используется для копирования элементов из исходного массива в целевой массив
Length Это свойство возвращает количество элементов в массиве
Rank Это свойство возвращает количество измерений в массиве
Reverse() Этот статический метод обращает содержимое одномерного массива
Sort() Этот статический метод сортирует одномерный массив внутренних типов. Если элементы в массиве реализуют интерфейс IComparer, то можно и спец типы.
```
Пример использования:
```csharp
string[] strs = {"Tone", "Bau", "Sis"};
foreach (string el in strs)
{
    WriteLine(el);
}
Array.Reverse(strs); //обратить массив
foreach (string el in strs)
{
    WriteLine(el);
}
Array.Clear(strs,1,2);
foreach (string el in strs)
{
    WriteLine(el);
}
ReadKey();
```


