# Стандартные интерфейсы

В библиотеке базовых классов платформы .NET определены стандартные интерфейсы, которые можно использовать при проектрировании своих классов.

## Итераторы базовые

Интерфейсы IEnumarable и IEnumerator используются в конструкциях foreach.

Интерфейс IEnumerable сообщает вызывающий код, что элементы этого объекта могут перечислятся:

```csharp
interface IEnumerable
{
    IEnumerator GetEnumerator();
}
```

А возвращаемый интерфейс IEnumerator позволяет вызывающему коду вызывать элементы контейнера:

```csharp
interface IEnumerator
{
    bool MoveNext();
    object Current { get; }
    void Reset();
}
```

Пример реализации в классе итератора по массиву:

```csharp
public class Sample : IEnumerable
{
    private int[] arr;
    public Sample()
    {
        arr = new int[] {19, 1, 4, 5, 5};
    }
    IEnumerator IEnumerable.GetEnumerator()
    {
        return arr.GetEnumerator();
    }
}
//использование
var els = new Sample();
foreach (var e in els)
{
    Console.WriteLine(e);
}
```

## Итераторы yield

Итератор — это член, который указывает, каким образом должны возвращаться внутренние элементы контейнера во время обработки в цикле foreach.

Пример обычного перебора:

```csharp
IEnumerator IEnumerable.GetEnumerator()
{
    foreach (var e in arr)
        yield return e;
}
```

Пример возвращения нескольких значений:

```csharp
IEnumerator IEnumerable.GetEnumerator()
{
    yield return 1;
    yield return 10;
    yield return 23;
    yield return 33;
}
```

## Интерфейс клонирования

В недрах System.Object существует метод MemberwiseClone() для получения поверхностной копии текущего объекта, этот метод - защищенный. Объект его может вызывать для клонирования себя.

Пример определения класса с поверхностным клонированием:

```csharp
public class Sample : ICloneable
{
    private int X { get; set; }
    public object Clone() => this.MemberwiseClone();
}
```

Использование:

```csharp
Sample s1 = new Sample();
Sample s2 = (Sample)s1.Clone();
```

Глубокое клонирование. Для того чтобы заставить метод Clone() создавать глубокую копию внутренних ссылочных типов класса, в составе которого есть ссылочные типы данных, нужно настроить объект, возвращаемый методом MemberwiseClone(). 

Если класс или структура содержат только типы значений, необходимо реализовать метод Clone() с использованием волшебного метода MemberwiseClone(), но если внутри типа есть ссылочный тип, тогда для глубокой копии нужно создать новый объект, который учитывает каждую переменную члена ссылочного типа:

```csharp
public class Sample : ICloneable
{
    public int X { get; set; }
    public string Name { get; set; }
    public object Clone()
    {
        Sample newSam = (Sample)this.MemberwiseClone();
        newSam.Name = this.Name;
        return newSam;
    }
}
```

Использование:

```csharp
Sample s1 = new Sample() {Name = "111", X = 2};
Sample s2 = (Sample)s1.Clone();
s2.Name = "22222";
s2.X = 1;
WriteLine($"{s1.Name} - {s1.X}");
WriteLine($"{s2.Name} - {s2.X}");
```

## Интерфейс сравнения

Для того, чтобы с помощью System.Array.Sost() можно было отсортировать массив из определенного пользователем типа нужно реализовать в этом типе интерфейс сравения.

Интерфейс сравнения IComparable позволяет объекту указывать, как он соотносится с другими подобными объектами. 

Пример реализации интерфейса в объекте:

```csharp
public class Sample : IComparable
{
    public int X { get; set; }
    public string Name { get; set; }
    int IComparable.CompareTo(object obj)
    {
        Sample temp = obj as Sample;
        if (temp != null)
        {
            if (this.X > temp.X)
                return 1;
            if (this.X < temp.X)
                return -1;
            else
                return 0;
        }
        else
        {
            throw new ArgumentException("Parameter is not a Sample");
        }
    }
}
```

Возвращаемые значения метода CompareTo:

```csharp
Любое число меньше нуля - этот экземпляр находится перед указанным объектом в порядке сортировки
Ноль - Этот экземпляр равен указанному объекту
Любое число больше нуля - этот экземпляр находится после указанного объекта в порядке сортировки
```

Более сокращенная реализация интерфейса сравнения:

```csharp
int IComparable.CompareTo(object obj)
{
    Sample temp = obj as Sample;
    if (temp != null)
    {
        return this.X.CompareTo(temp.X);
    }
    else
        throw new ArgumentException("Parameter is not a Sample");
}
```

Использование:

```csharp
Sample[] arrs =
{
    new Sample {Name = "And", X = 1},
    new Sample {Name = "Two", X = 2}, 
};
Array.Sort(arrs);
foreach (var elem in arrs)
{
    WriteLine($"{elem.Name} - {elem.X}");
}
```

Вожможно простроение реализации метода интерфейса сравнения, использующего поля класса объекта с помощью интерфейса IComparer. 

Для этого потребуется создать дополнительный класс, реализующий интерфейс IComparer:

```csharp
public class SampleComparer : IComparer
{
    int IComparer.Compare(object o1, object o2)
    {
        Sample s1 = o1 as Sample;
        Sample s2 = o2 as Sample;
        if (s1 != null && s2 != null)
            return String.Compare(s1.Name, s2.Name);
        else
            throw new ArgumentException("Parameter is not a Sample!");
    }
}
```

Теперь использование этого класса в сортировке:

```csharp
Array.Sort(arrs, new SampleComparer()); //сортировка
```

