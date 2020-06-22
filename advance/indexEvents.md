# События

Прямая работа с делегатами может приводить к написанию стереотипного кода (определение делегата, определение необходимых переменных-членов, создание спец методов регистрации и отмены регистрации). Во время применения делегатов как публичных типов в классах могут возникнуть всякие неприятные проблемы. Практика предусматривает объявление переменных-членов типа делегатов, только как закрытых.

Для избавления от необходимости добавления методов добавления и удаления методов из списка делегата, в языке C# предлагается event. В результате использования этой технологии класс автоматически получает методы регистрации и отмены регистрации, а также все необхожимые члены для типов делегатов, а такие переменные типа делегатов всегда будут закрытые и недоступны извне.

Процесс объявления события:

1. определить тип делегата, который будет хранить список методов, подлежащих вызову при возникновении события.

2. объявить событие (event) в терминах связанного типа делегата.

Когда компилятор C# обрабатывает ключевое слово event, он генерирует два скрытых метода, один с префиктом add_, а другой с префиксом remove_. За префиксом следует имя события C#.

>События - способ сэкономить время на наборе кода.

Пример объявления в классе события:
```csharp
class MyClass
{
    public delegate void Handler(string message); //тип делегата
    public event Handler Message; //событие - к нему регистрироватся
    public void Action(string addMess) //действие которое вызывает другой метод
    {
        Message?.Invoke($"Message for {addMess}!"); //Здесь можно применять null-условную операцию. Это инициирование событие.
    }
}
```
Пример использования события:
```csharp
static void Main()
{
    MyClass myObject = new MyClass();
    MyClass.Handler myDelegate = Message; //делегат
    myObject.Message += myDelegate; //регистрация на источник события
    myObject.Action("User"); //Message method => Message for User!
    myObject.Message -= myDelegate; //отключение от источника события
    myObject.Action("Admin"); 
    myObject.Message += Message; //упрощенный метод с применением группового преобразования методов
    myObject.Action("Fast Method"); //Message method => Message for Fast Method!
    ReadKey();
}
static void Message(string message)
{
    WriteLine($"Message method => {message}!");
}
```
>Можно применять упрощенную регистрацию событий с использованием Visual Studio. Нужно набрать += и нажать кнопку <Tab> для автоматического завершения связанного экземпляра делегата. Это доступно благодаря синтаксису групповых преобразований методов.

Еще пример применения механизма событий через инкапсуляцию - подписка / отписка:
```csharp
class MyUser
{
    public string Name { get; set; }
    public MyUser(string name)
    {
        Name = name;
    }
    public void Say(string mes)
    {
        Console.WriteLine($"{Name} сказал {mes}");
        eventSay?.Invoke(this, mes);
    }
    //событие - что-нить сказать
    private Action<object, string> eventSay;
    public event Action<object, string> EventSay
    {
        add
        {
            Console.WriteLine($"{Name} стали слушать");
            eventSay += value;
        }
        remove
        {
            Console.WriteLine($"{Name} перестали слушать");
            if (eventSay != null) eventSay -= value;
        }
    }
    public void PrintHears()
    {
        foreach (var el in eventSay.GetInvocationList())
        {
            if (el.Target is MyUser user)
            {
                Console.WriteLine(user.Name);
            }
        }
    }
    public void Hear(object obj, string mes)
    {
        var user = obj as MyUser;
        Console.WriteLine($"Человек {Name} услышал >>> {user.Name} сказал {mes}");
    }
}
class Program
{
    static void Main(string[] args)
    {
        MyUser user1 = new MyUser("Федя");
        user1.Say("Привет мир!");
        Console.WriteLine();
        MyUser user2 = new MyUser("Петя");
        user1.EventSay += user2.Hear;
        user1.Say("Еще раз привет!");
        MyUser user3 = new MyUser("Дима");
        user1.EventSay += user3.Hear;
        Console.WriteLine("\nСлушают первого:");
        user1.PrintHears();
        Console.ReadKey();
    }
}
```

## Специальные аргументы событий

Можно использовать спец аргументы передаваемых аргуметов событию. Для того, чтоб передать специальные данные нужно построить подходящий класс, производный от EventArgs.

Пример класса-сообщения, который поддерживает строковое представление сообщения:
```csharp
class MyEventArgs : EventArgs //класс передачи информации - строкового сообщения
{
    public readonly string msg;
    public MyEventArgs(string message) => msg = message;
}
```
Пример объявления класса с использованием этого класса-сообщения:
```csharp
class MyClass
{
    public string name = "User";
    public delegate void Handler(object sender, MyEventArgs e);
    public event Handler Message;
    public void Action(string addMess)
    {
        Message?.Invoke(this, new MyEventArgs(addMess)); //инициировать событие
    }
}
```
Вызов событий используя этот измененный класс.
```csharp
static void Main()
{
    MyClass myObject = new MyClass();
    myObject.Message += Message; //упрощенный метод с применением группового преобразования методов
    myObject.Action("Fast Method");  //Message method => Message for Fast Method!!
    ReadKey();
}
static void Message(object sender, MyEventArgs e)
{
    WriteLine($"{sender} method => {e.msg}!"); //вывод сообщения через поле, доступное только для чтения
    if (sender is MyClass c) //можно взаимодействовать с объектом
    {
        WriteLine($"Name in class = {c.name}"); //вывод свойства из полученного объекта
    }
}
```

## Использование обобщенного делегата EventHandler<T>
    
Можно использовать обобщенный делегат, который принимает экземпляр object в первом параметре и экземпляр производного от EventArgs класса во втором. В нем T - специальный тип, производный от EventArgs. 

Нужно также объявлять класс - сообщение, который поддерживает строковое представление сообщения:
```csharp
class MyEventArgs : EventArgs //класс передачи информации - строкового сообщения
{
    public readonly string msg;
    public MyEventArgs(string message) => msg = message;
}
```
Пример объявления класса с использованием этого класса-сообщения:
```csharp
class MyClass
{
    public event EventHandler<MyEventArgs> Message; //обявление обобщенного события
    public void Action(string addMess)
    {
        Message?.Invoke(this, new MyEventArgs(addMess)); //инициировать событие
    }
}
```
Пример применения этого класса:
```csharp
static void Main()
{
    MyClass myObject = new MyClass();
    myObject.Message += Message; //упрощенный метод с применением группового преобразования методов
    myObject.Action("Fast Method");  //Message method => Message for Fast Method!!
    myObject.Message -= Message;
    EventHandler<MyEventArgs> deleg = new EventHandler<MyEventArgs>(Message);
    myObject.Message += deleg; //регистрация метода с применением старого способа
    myObject.Action("Old Method");
    ReadKey();
}
static void Message(object sender, MyEventArgs e)
{
    WriteLine($"{sender} method => {e.msg}!"); //вывод сообщения через поле, доступное только для чтения
}
```

## Анонимные методы
В языке C# событие можно ассоциировать прямо с блоком операторов кода во время регистрации события. Формально такой код называется анонимным методом. Демонстрация применения анонимного метода на основе выше употребимого класса:
```csharp
static void Main()
{
    MyClass myObject = new MyClass();
    myObject.Message += delegate //анонимный метод
    {
        WriteLine("Oook!");
    };
    myObject.Message += delegate(object sender, MyEventArgs args) //анонимный метод с параметрами
    {
        WriteLine($"Message => {args.msg}");
    };
    myObject.Message += (sender, args) 
        => WriteLine($"Message2 => {args.msg}"); //анонимный метод в виде лямбды
    myObject.Action("Username"); //Oook! \n Message => Username \n Message2 => Username
    ReadKey();
}
static void Message(object sender, MyEventArgs e)
{
    WriteLine($"{sender} method => {e.msg}!"); //вывод сообщения через поле, доступное только для чтения
}
```
Доступ к локальным переменным из анонимного метода имеет интересные особенности:

- анонимный метод не имеет доступа к параметром ref и out определяющего метода.

- анонимный метод не может иметь локальную переменную, имя которой совпарадет с имененм локальной переменной внешнего метода.

- анонимный метод может обращаться к переменным экземлпяра (или статистическим перменным) из области действия внешнего класса.

- анонимный метод может объявлять локальную переменную с тем же именем, что и у переменной-члена внешнего класса.

## Лямбда-выражения

Лямбда выражения могут использоваться везде, где должен применяться анонимный метод или строго типизированный делегат. Компилятор C# транслирует лямбда-выражения в стандартный анонимный метод, использующий стандартный тип делегата.

Лямбда-выражения можно представить следующим образом:
```csharp
АргументыДляОбработки => ОбрабатывающиеОператоры
```
Оператор кода лямбда-выражения:
```csharp
List<int> evenList = list.FindAll(i => (i % 2) == 0);
```
компилируется в такой код C#:
```csharp
List<int> evenList = list.FindAll(i =>
{
    return (i % 2) == 0;
});
```
Язык C# позволяет строить лямбда-выражения, сосоящие из множества операторов. Нужно обозначить для этого блок с помощью фигурных скобок. Пример:
```csharp
List<int> evenList = list.FindAll(i =>
{
    WriteLine("value is one");
    bool isEven = (i % 4) == 0;
    return isEven;
});
```
Лямбда-выражения могут быть быть с множеством параметров:
```csharp
(int one, string msg) => WriteLine($"{one} => {msg}")
```
Для перехвата всех событий, поступающих от этого объекта, может применяться синтаксис лямбда-выражений:
```csharp
static void Main()
{
    MyClass myObject = new MyClass();
    myObject.Message += (sender, e) => 
    {
        WriteLine(e.msg);
    }; //регистрация с помощью лямбда-выражания
    myObject.Action("Old Method");
    ReadKey();
}
```

## Применение лямбда-выражений, сжатых до выражений

В C# версии 6 появилась возможность использовать операцию => для упрощения реализаций членов:
```csharp
myObject.Message += (sender, e) =>  WriteLine(e.msg); //регистрация лямбда-выражанием
```
В C# версии 7 можно применять синтаксис для конструкторов, финализаторов, для средств доступа get и set к свойствам:
```csharp
class MyClass
{
    private string name;
    public MyClass(string name) => this.name = name;
    public string Name
    {
        get => name;
        set => name = value;
    }
}
```

