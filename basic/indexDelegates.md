# Делегаты

Делегат в C# по сути является объектом, который может ссылаться на метод. Когда создается делегат, мы получаем объект, в котором есть ссылка не метод. Метод можно вызывать по этой ссылке. Делегат позволяет вызвать метод, на который тот ссылается.

Делегат - это безопасный в отношении типов объект, указывающий на другой метод (или список методов) приложения, который может быть вызван позднее в коде. Как только делегат создан, снабжен информацией, он может во время выполнения приложения динамически вызввать методы, на которые указывает.

Делегат поддерживает три важных порции информации (или свойства):

- адрес метода, к которому он делает вызовы;

- аргументы (если есть) вызываемого метода;

- возвращаемое значение (если есть) вызываемого метода.

После того, как делегат создан и снабжен необходимой информацией, он может во время выполнения динамическки вызывать метод или методы, на которые указывает. 

>Каждый делегат имеет способность вызывать свои методы как синхронно, так и асинхронно.

>Делегаты могут указывать либо на статические методы, либо на методы экземпляра.

## Определение делегата

Определение делегата без параметров:
```csharp
delegate void Fun();
```
Определение делегата с сигнатурой - эта же сигнатура должна быть у других методов или метода, на которые он будет указывать.
```csharp
public delegate int BinOp(int x, int y);
```
Определение делегата может содержать наборы аргументов in out ref params.
```csharp
delegate int Godus(out bool a, ref bool b, in bool c, int d, params int[] arrs);
```
Определение типа делегата C# дает в результате запечатанный класс с тремя сгенерированными компилятором методами, в которых типы параметров и возвращаемые типы основаны на объявлении делегата.

Когда используется ключевое слово delegate - то тем самым неявно создается класс, который является MulticastDelegate, а тот в свою очередь - Delegate.

Некоторые члены этого класса:
```csharp
Член                Назначение
Method              Свойство возвращает объект System.Reflection.Method, который представляет детали статического метода, поддерживаемого делегатом
Target              Если подлежащий вызову метод нестатический - то он возвращает объект, который представляет метод, поддерживаемый делегатом. Если null - то метод статический.
Combine()           Этот статический метод добавляет метод в список, поддерживаемый делегатом - тоже самое что и +=.
GetInvocationList() Этот метод возвращает массив объектов System.Delegate, каждый из которых представляет определенный метод, доступный для вызова.
Remove()RemoveAll() Эти статические мтоды удаляют методы или все методы из списка вызовов делегата - то же что и -=.
```

Пример использования:
```csharp
delegate int BinOp(int x, int y);
public class MyClass
{
    public static int Add(int x, int y) => x + y;
    public static int Subtract(int x, int y) => x - y;
}
static void Main(string[] args)
{
    BinOp deleg = new BinOp(MyClass.Add);
    WriteLine(deleg(10, 10)); //20
    WriteLine(deleg.Invoke(20,20)); //40
    ReadKey();
}
```
Еще пример использования:
```csharp
public delegate int MyFunc(int x); //описание делегата
public static void PrintMyFunc(MyFunc myFunc, int x) //метод с делагатом в параметрах
{
    int y = myFunc(x); //использование делегата - вызов метода, на который он ссылается
    Console.WriteLine($"Результат y = {y}");
}

public static int PlusPlus(int x) => x++; //просто функция
static void Main(string[] args)
{
    int i = 10;
    MyFunc variablDeleg = PlusPlus; //объявление объекта делегат
    PrintMyFunc(variablDeleg, i); //вызов метода с делегатом в параметрах
    Console.ReadKey();
}
```
>Делегаты в .NET безопасны в отношении типов, если попытаться передать делегату метод, не соответствующий его шаблону, то можно получить ошибку на этапе комплияции.

## Статический и нестатический метод в делегате

Статический метод:
```csharp
delegate int BinOp(int x, int y);

public static class MyStaticClass
{
    public static int Add(int x, int y) => x + y;
}

static void DisplDelegInfo(Delegate delObj)
{
    foreach (Delegate el in delObj.GetInvocationList())
    {
        WriteLine($"Method name = {el.Method}");
        WriteLine($"Type = {el.Target}");
    }
}

static void Main(string[] args)
{
    BinOp deleg = new BinOp(MyStaticClass.Add); //статический метод
    DisplDelegInfo(deleg); // Method name = Int32 Add(Int32, Int32)        Type =     
    ReadKey();
}
```
Нестатический метод:
```csharp
delegate int BinOp(int x, int y);

public class MyClass
{
    public int Add(int x, int y) => x + y;
}

static void DisplDelegInfo(Delegate delObj)
{
    foreach (Delegate el in delObj.GetInvocationList())
    {
        WriteLine($"Method name = {el.Method}");
        WriteLine($"Type = {el.Target}");
    }
}

static void Main(string[] args)
{
    MyClass my = new MyClass();
    BinOp del2 = new BinOp(my.Add); //нестатический метод
    DisplDelegInfo(del2); // Method name = Int32 Add(Int32, Int32)          Type = ConsoleApp4.Program+MyClass
    ReadKey();
}
```

## Отправка уведомлений о состоянии объекта

1. Определить новый тип делегата, который будет использоваться для отправки уведомлений вызывающему коду.

2. Объявить переменную-член этого типа делегата в классе Bas.

3. Создать в классе Bas вспомогательную функцию, которая позволяет вызывающему коду указывать метод для обратного вызова.

4. Реализовать метод Action() для обращения к списке вызовов делегата в подходящих обстоятельствах.

Первые всех действий в классе:
```csharp
public class Bas
{
    // 1. определить тип делегата
    public delegate void BasHandler(string msgForCaller);
    // 2. определить переменную-член этого типа делегата
    private BasHandler listHandlers;
    // 3. добавить регистрационную функцию для вызывающего кода
    public void RegisterBusHandler(BasHandler methodHandlerCall)
    {
        listHandlers = methodHandlerCall;
    }
    // 4. Реализовать метод Action()
    public void Action(bool b)
    {
        if (b)
            if (listHandlers != null)
                listHandlers("Hello! It is true!");
    }
}
static void Main(string[] args)
{
    Bas bas = new Bas();
    bas.RegisterBusHandler(new Bas.BasHandler(OnBasEvent));
    bas.Action(true); //Message = Hello
    ReadKey();
}
public static void OnBasEvent(string msg)
{
    Console.WriteLine($"Message = {msg}");
}
```

## Включение группового вызова и удаление целей из списка
```csharp
delegate void MultiDeleg(string msgForCaller);
static void Main(string[] args)
{
    MultiDeleg multi = OnBasEvent;
    multi += OnBasEvent;
    multi.Invoke("One"); //Mess One   Mess One
    multi -= OnBasEvent;
    multi.Invoke("Two"); //Mess Two
    ReadKey();
}
public static void OnBasEvent(string msg)
{
    Console.WriteLine($"Message = {msg}");
}
```





# Продвинутое

## Обобщенные делегаты
Язык C# позволяет определять обобщенные типы делегатов. Пример определения обобщенного типа делегата.
```csharp
public delegate void MyGenDelegate<T>(T arg);//обобщенный делегат
static void Main(string[] args)
{
    MyGenDelegate<int> strTarget = new MyGenDelegate<int>(IntTarget); //регистрация цели
    strTarget.Invoke(10);
    MyGenDelegate<int> str2Target = IntTarget; //регистрация цели
    str2Target(10);
    ReadKey();
}
static void IntTarget(int arg) //цель делегата
{
    WriteLine($"arg is: {arg}");
}
```

## Обобщенные делегаты Action<> и Func<>
В C# есть встроенные в инфраструктуру делегаты Action<> и Funс<>. Их можно исползовать когда просто необходим какой нибудь делегат, который принимает набор аргументов и возможно возвращает значение, отличное от void.

Обобщенный делегат Action<> можно использовать для указания на метод, который принимает до 16 аргументов и возвращает void.

Пример Action<>:
```csharp
static void DisplMessage(string msg) //цель для обобщенного делегата
{
    WriteLine($"Message: {msg}");
}
static void Main(string[] args)
{
    Action<string> actionTarget = DisplMessage; //делегат с указанием на метод
    actionTarget.Invoke("Hello!");
    ReadKey();
}
```
Обобщенный делегат Func<> можно использовать для указания на методы, которые, подобно Action<>, принимают вплоть до 16 параметров и имеют специальное возвращаемое значение, при этом следует учесть, что последний параметр в Func<> всегда показывает возвращаемое значение метода.

Пример Func<>:
```csharp
static int Add(int x, int y) //цель для обобщенного делегата
{
    return x + y;
}
static void Main(string[] args)
{
    Func<int, int, int> funcTarget = Add; //делегат с указанием цели
    int result = funcTarget.Invoke(10, 10);
    WriteLine($"Result = {result}");
    ReadKey();
}
```

