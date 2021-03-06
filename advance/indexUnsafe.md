# Небезопасный код

## Указатели

В рамках платформы определены дополнительно типы указателей. Список допустимых операций и ключевых операций, связанных с указателями:

```csharp
Операция            Назначение
*                   Создание переменной указателя. Как и в языке C++, та же самая операция используется для разыменовывания указателей.
&                   Применяется для получения адреса переменной в памяти.
->                  Применяется для доступа к полям типа, представленного указателем.
[]                  В небезопасном коде позволяет индексировать области памяти, на которую указывает переменная указателя.
++ --               Могут применяться к типам указателей
+ -                 Могут применяться к типам указателей
== != < > <= >=     Могут применяться к типам указателей
stacalloc           Применяется для размещения массивов C# прямо в стеке
fixed               Применяется для временного закрепления переменной, чтобы потом можно найти адрес
```

## Ключевое слово небезопасного кода
Для работы с указателями применяется блок небезопасного кода с ключевым словом unsafe. 

Пример такого блока:
```csharp
class Program
{
    static void Main()
    {
        unsafe
        {
        }
    }
}
```
Можно строить структуры небезопасными:
```csharp
unsafe struct Node
{
    public int Value;
}
```
Методы также могут быть небезопасными:
```csharp
static unsafe void Sample(int* intPointer)
{
    *intPointer *= *intPointer;
}
```
Использование небезопасного кода:
```csharp
unsafe
{
    int i = 10;
    Sample(&i);
    WriteLine($"Val = {i}");
    ReadKey();
}
//или
static unsafe void Main()
{
    int i = 10;
    Sample(&i);
    WriteLine($"Val = {i}");
    ReadKey();
}
```
## Работа с операциями * и &.

Послу установки небезопасного блока можно строить указатели с помощью * и получать адрес указываемых данных &. Но в отличие от C++ операция * применяется только к лежащему в основе типу, а не префиксом к имени каждой переменной указателя.

Пример:
```csharp
int* pi, pj;
```
Пример кода:
```csharp
static unsafe void Sample(int* intPointer)
{
    *intPointer *= *intPointer;
}
//первый пример
static void Main()
{
    int myInt = 10;
    unsafe
    {
        int* pInt = &myInt;
        *pInt = 12;
        Sample(pInt);
        WriteLine($"Val = {myInt}");
    }
    ReadKey();
}
//второй пример
static unsafe void Main()
{
    int myInt = 10;
    int* pInt = &myInt;
    *pInt = 12;
    Sample(pInt);
    WriteLine($"Val = {myInt}");
    ReadKey();
}
//другой пример
static void Main()
{
    int myInt = 10;
    unsafe
    {
        Sample(&myInt);
    }
    WriteLine($"Val = {myInt}");
    ReadKey();
}
```

## Доступ к полям через указатели ->

В небезопасном блоке -> является небезопасной версией стандартной (безопасной) операции точки (.). Используя операцию обращения к указателю * можно разыменовывать указатель для применения операции точки.
```csharp
static unsafe void UsePoiunts()
{
    Node node;
    Node* n = &node;
    n->Value = 10;
    WriteLine($"Val = {n->Value} and {n->ToString()}");
}
```
## Локальная переменная в стеке

С использованием ключевого слова stackalloc можно выделить паять для переменной непосредственно в стеке, и поэтому она не обрабатывается сборщиком мусора.
```csharp
static unsafe void UnsafeStack()
{
    char* p = stackalloc char[256];
    for (int i = 0; i < 256; i++)
    {
        p[i] = (char)i;
    }
}
```
## Фиксация переменной ссылочного типа в куче

Использованием ключевого слова fixed можно в памяти из небезопасного контекста фиксировать переменную в памяти. Без fixed от указателей на управляемые переменные было бы мало толку, поскольку сборщик мусора может пермешать переменные в памяти непредсказуемым образом.

Ключевое слово fixed позволяет строить оператор, который фиксирует ссылочную переменную в памяти, чтобы ее адрес оставался постоянным на протяжении работы оператора. Каждый раз при взаимодействии со ссылочным типом из небезопасного кода нужно закрепить ссылку.
```csharp
class Node
{
    public int x;
}
static unsafe void UseAndPin()
{
    Node node = new Node
    {
        x = 10,
    };
    fixed (int* p = &node.x)
    {
        *p = 10;
    }
}
```





