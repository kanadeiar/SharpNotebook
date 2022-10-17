# Перечислимые типы

## Перечисления

Перечислимый тип - это набор пар, состоящих из символьных названий и значений. 

Перечислимый тип напрямую наследуется от типа System.Enum, производный от System.ValueType, а тот, в свою очередь - от System.Object. Перечислимые типы относятся к значимым типам и могут существовать как в упакованном, так и в неупакованном виде.

Пример определения перечисления:

```csharp
internal enum Color
{
    Red
    Green,
    Blue,
}
```

Программу с использованием перечислимых типов проще написать и сопровождать. Перечислимые типы подвергаются строгой проверке типов.

У перичислимого типа не может быть методов и свойств, но можно использовать методы расширения.

Перичислимый тип - это структура, внутри которой описан набор константных полей и одно экземплярное поле.

Эти методы позволяют получить тип, используемый для хранения значения в перечислении:

```csharp
public static Type GetUnderlyingType(Type enumType); // Определен в типе System.Enum
public Type GetEnumUnderlyingType(); // Определен в типе System.Type
Console.WriteLine(Enum.GetUnderlyingType(typeof(Color)));
Console.WriteLine(Enum.Format(typeof(Color), 3, "G"));
```

В основе любого перечисления лежит один из основных примитивных целочисленных типов. 

```csharp
internal enum Color : byte
{
    Red = 10
    Green = 1,
    Blue,
}
```

Вывод данных по перечислению:

```csharp
var colors = (Color[])Enum.GetValues(typeof(Color));
Console.WriteLine("Count = " + colors.Length);
foreach (var c in colors)
{
    Console.WriteLine("{0:D}\t{0:G}", c);
}
```

Метод из библиотеки Kanadeiar.Core возвращающий массив данных со значениями перечисления:

```csharp
var colors = Tools.GetEnumValues<Color>();
```

Есть другие методы для получения символических имен перечислимых типов, а также для получения значения перечислимого типа на основе строкового значения и еще для проверки на допустимость значения перечисления:

```csharp
var c1 = Color.Black;
var name = Enum.GetName(typeof(Color), c1);
name = typeof(Color).GetEnumName(c1);
var names = Enum.GetNames(typeof(Color));
names = typeof(Color).GetEnumNames();
var c2 = (Color)Enum.Parse(typeof(Color), "red", true);
Enum.TryParse<Color>("1", true, out var c3);
Console.WriteLine($"{name} {c2} {c3}");
Console.WriteLine(Enum.IsDefined(typeof(Color), 1));
Console.WriteLine(Enum.IsDefined(typeof(Color), "Red"));
Console.WriteLine(Enum.IsDefined(typeof(Color), 30));
Console.WriteLine(Enum.IsDefined(typeof(Color), "red"));
```

Перечислимые типы всегда применяют рядом с другими типами, обычно в качестве параметров методов или возвращаемых значений из полей, свойств. Обычно перечислимый тип определяется на уровне класса, где он используется. Лучше определять перечисление на уровне с основным классом, где он будет использоваться.

## Битовые флаги

Побитовые операции предлагают быстрый механизм для работы с двоичными числами на уровне битов.

Пример работы с битовыми операциями:

```csharp
Console.WriteLine("6 & 4 = {0} | {1}", 6 & 4, Convert.ToString(6 & 4,2));
Console.WriteLine("6 | 4 = {0} | {1}", 6 | 4, Convert.ToString(6 | 4,2));
Console.WriteLine("6 ^ 4 = {0} | {1}", 6 ^ 4, Convert.ToString(6 ^ 4,2));
Console.WriteLine("6 << 1  = {0} | {1}", 6 << 1, Convert.ToString(6 << 1,2));
Console.WriteLine("6 >> 1 = {0} | {1}", 6 >> 1, Convert.ToString(6 >> 1,2));
Console.WriteLine("~6 = {0} | {1}", ~6, Convert.ToString(~6, 2));
Console.WriteLine("Int.MaxValue {0}", Convert.ToString(int.MaxValue, 2));
```

При создании набора комбинируемых друг с другом битовых флагов используют перечислимые типы. Определяя перечислимый тип, предназначенный для идентификации битовых флагов, следует явно присвоить каждому имени значение. Каждому идентификатору - 1 бит. Идентификатору None - обычно значение 0. Рекомендуется устанавливать аттрибут FlagsAttribute. 

Определение битовых флагов и их использование:

```csharp
[Flags]
enum Actions
{
    None = 0,
    Read = 0x0001,
    Write = 0x0002,
    ReadWrite = Actions.Read | Actions.Write,
    Delete = 0x0004,
    Query = 0x0008,
    Sync = 0x0010
}
Actions actions = Actions.Delete | Actions.Read;
Console.WriteLine(actions);
```

Форматируя экземпляр перечислимого типа с использованием общего формата, метод ToString ищет атрбут FlagsAttribute. Если его нет - вертается идентификатор, соответствующий числовому значению. Если этот атрибут есть, тогда CLR: 

- получает набор числовых значений, сортирует их;

- для каждого значения выполняется операция коньюнкции - AND с экземпляром перечисления, и в случае равенства, строка добавляется к возвращаемому значению;

- если исходное значение перечисления не равно нулю, метод возвращает набор сиволов, разделенных запятой;

- если исходным значением перечисления является 0, возвращается это значение;

- если ничего не найдено, то возвращается значение 0.

Идентификаторы, определенные в перечислении не обязаны быть степенью двойки.

```csharp
Actions actions = Actions.Delete | Actions.Read;
Console.WriteLine(actions);
Console.WriteLine(actions.ToString("F"));
var a = (Actions)Enum.Parse(typeof(Actions), "query", true);
Console.WriteLine(a.ToString());
Enum.TryParse(typeof(Actions), "query, read", true, out var a2);
Console.WriteLine(a2?.ToString());
var a3 = Enum.Parse(typeof(Actions), "28");
Console.WriteLine(a3.ToString());
```

В методах Parse и TryParse производятся следующие операции:

- удаляются все пробелы в начале и конце строки;

- если первый символ - плюс или минус, на входе число, возвращается экземпляр, равный переданному числу;

- переданная строка разбивается на разделенные запятыми лексемы, у каждой удаляются пробелы в начале и конце;

- выполняется поиск каждой строки лексемы среди перечисления, и если не найдется, то генерируется исключение или вертается false;

- после обнаружения и проверки всех лексем результат вертается из метода.

Нельзя использовать метод IsDefined с перечислениями-флагами - результат поиска будет нулевым.

## Методы перечислимых типов

Использованием методов расширения перечислимых типов можно расширить функциональность перечислений.

```csharp
public static class Extensions
{
    public static bool IsSet(this Actions flags, Actions flagToTest)
    {
        if (flagToTest == 0)
            throw new ArgumentOutOfRangeException(nameof(flagToTest));
        return (flags & flagToTest) == flagToTest;
    }
    public static bool IsClear(this Actions flags, Actions flagToTest)
    {
        if (flagToTest == 0)
            throw new ArgumentOutOfRangeException(nameof(flagToTest));
        return !IsSet(flags, flagToTest);
    }
    public static bool AnyFlagsSet(this Actions flags, Actions testFlags)
    {
        return ((flags & testFlags) != 0);
    }
    public static Actions Set(this Actions flags, Actions setFlag)
    {
        return (flags | setFlag);
    }
    public static Actions Clear(this Actions flags, Actions clearFlags)
    {
        return (flags | ~clearFlags);
    }
    public static void ForEach(this Actions flags, Action<Actions> processFlag)
    {
        if (processFlag == null)
            throw new ArgumentNullException(nameof(processFlag));
        for (uint bit = 1; bit != 0; bit <<= 1)
        {
            uint temp = ((uint)flags) & bit;
            if (temp != 0)
            {
                processFlag.Invoke((Actions)temp);
            }
        }
    }
}
// Использование
Actions actions = Actions.Delete | Actions.Read;
var bb = actions.IsSet(Actions.Read);
actions = actions.Set(Actions.Sync);
actions = actions.Clear(Actions.Read);
```


