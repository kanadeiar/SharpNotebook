## Сериализация (XML, настройка)

Термин «сериализация» описывает процесс сохранения состояния объекта в потоке (например, файловом потоке или потоке в памяти). Последовательность сохраняемых данных содержит всю информацию, необходимую для реконструкции (или десериализации) состояния объекта с целью последующего использования. 

Двоичная сериализация - BinaryFormatter.

SOAP сериализация - SoapFormatter.

XML сериализация - XmlSerializer.

Пример объявления класса с возможностью сериализации:
```csharp
[Serializable]
class MyClass
{
    public string Name { get; set; }
    public int Size { get; set; }
}
```
Пример сохранения этого объекта в компактном двоичном формате:
```csharp
class Program
{
    static void Main()
    {
        MyClass myClass = new MyClass
        {
            Name = "Первый",
            Age = 10,
        };
        BinaryFormatter binary = new BinaryFormatter(); //сохранение в компактном двоичном формате
        using (FileStream fstream = new FileStream("test.dat",FileMode.Create,FileAccess.Write,FileShare.None))
        {
            binary.Serialize(fstream, myClass);
        }
        Console.WriteLine("Нажмите любую кнопку для выхода...");
        Console.ReadKey();
    }
}
```

Для сериализации нужно пометить класс атрибутом [Serializable]. Если в этом классе какой-то тип или член-данные не должны сохранятся, тогда их нужно пометить атрибутом [NonSerialize], это нужно для членов, которые не нужно запоминать. Атрибут [Serializable] не наследуется от родительского класа, порожденный класс от типа, помеченного [Serializable], тоже должен быть помечен этим атрибутом, наче он не может быть быть сохранен в потоке.

Пример объявления классов с возможностью сериализации:
```csharp
[Serializable]
class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
    [NonSerialized] public string Secret = "XF-1244";
}
[Serializable]
class Home
{
    public Person Person { get; set; } = new Person();
    public int Size { get; set; }
}
[Serializable]
class BigHome : Home
{
    public int Count { get; set; }
}
```

Способы сериализации:

BinaryFormatter сериализукт состояние объекта в поток, используя компактный двоичный формат.

SoapFormatter сохраняет состояние объекта в виде сообщения SOAP (XML-формат для передачи и приема сообщений от веб-служб, основанных на SOAP).

XmlSerializer сохраняет объекты в документе XML.

С использованием типа BinaryFormatter или SoapFormatter сохранение объекта происходит всех сериализуемых полей типа независимо от того, являются они открытыми, закрытыми, или закрытыми доступными через открытые члены, все кроме [NonSerialized].

С использованием XmlSerializer сохранение объекта в XML файл будет только открытых полей или закрытых данных, доступных посредством открытых свойств. Закрытые и не доступные через свойства и [NonSerialized] - игнорируются.

Эти способы отличаются точностью воспроизведения типов. BinaryFormatter сохраняет не только данные полей объектов из графа, но также полностью заданное имя каждого типа и полное имя определяющей его сборки (имя, версию, маркер открытого ключа, культуру), это нужно например при передаче объектов по значению - полные копии - между границами машин в приложениях .NET. SoapFormatter лишь сохраняет следы сборки источника на основе XML. А XmlSerializer не пытается сберечь точную информацию о типе, он не записывает полностью заданное имя типа или сборку, где тип определен, это полезно например, там где нужно чтоб с объектом можно было работать из разных операционных систем, платформ, языков программирования.

## Двоичная сериализация

Пример использования двоичной сериализации:
```csharp
[Serializable]
class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
    [NonSerialized] public string Secret = "XF-1244";
}
[Serializable]
class Home
{
    public Person Person { get; set; } = new Person();
    public int Size { get; set; }
}
[Serializable]
class BigHome : Home
{
    public int Count { get; set; }
}
static void Main()
{
    SaveBinary();
    ReadNPrintBinary();
    Console.WriteLine("Нажмите любую кнопку для выхода...");
    Console.ReadKey();
    void SaveBinary()
    {
        Person person = new Person
        {
            Name = "Тестирование", 
            Age = 22, 
            Secret = "X=2213"
        };
        BigHome home = new BigHome
        {
            Person = person,
            Size = 10,
            Count = 1,
        };
        BinaryFormatter binFormatter = new BinaryFormatter();
        using (FileStream stream = new FileStream("data.dat",FileMode.Create,FileAccess.Write,FileShare.None))
        {
            binFormatter.Serialize(stream, home);
        }
        Console.WriteLine("Объект сохранен в бинарном формате...");
    }
    void ReadNPrintBinary()
    {
        BinaryFormatter binFormatter = new BinaryFormatter();
        using (FileStream stream = File.OpenRead("data.dat"))
        {
            BigHome home = (BigHome)binFormatter.Deserialize(stream);
            Console.WriteLine($"Прочитанный объект: {home.Person.Name}, {home.Person.Age}, {home.Size}, {home.Count}");
        }
    }
}
```

## SOAP сериализация

Пример использования SOAP сериализации:
```csharp
//теже объекты что и выше
static void Main()
{
    SaveSoap();
    ReadNPrintSoap();
    Console.WriteLine("Нажмите любую кнопку для выхода...");
    Console.ReadKey();
    void SaveSoap()
    {
        Person person = new Person
        {
            Name = "Тестирование", 
            Age = 22, 
            Secret = "X=2213"
        };
        BigHome home = new BigHome
        {
            Person = person,
            Size = 10,
            Count = 1,
        };
        SoapFormatter soapFormatter = new SoapFormatter();
        using (FileStream stream = new FileStream("data.soap",FileMode.Create,FileAccess.Write,FileShare.None))
        {
            soapFormatter.Serialize(stream, home);
        }
        Console.WriteLine("Объект сохранен в SOAP формате...");
    }
    void ReadNPrintSoap()
    {
        SoapFormatter soapFormatter = new SoapFormatter();
        using (FileStream stream = File.OpenRead("data.soap"))
        {
            BigHome home = (BigHome)soapFormatter.Deserialize(stream);
            Console.WriteLine($"Прочитанный объект: {home.Person.Name}, {home.Person.Age}, {home.Size}, {home.Count}");
        }
    }
}
```


## XML сериализация

Некоторые атрибуты из пространства Serialization:
```csharp
Член                    Описание
[XmlAttribute]          Атрибут .NET можно применять к полю или свойство для сообещния о необходимости сериалиовать данные как атрибут XML а не как подэлемент
[XmlElement]            Поле или свойство будет сериализовано как элемент XML, именованный по выбору
[XmlEnum]               Атрибут предоставляет имя элемента члена перечисления
[XmlRoot]               Атрибут управляет тем, как будет сконструирован корневой элемент
[XmlText]               Свойство или поле будет сериализовано как текст XML
[XmlType]               Аррибут предоставляет имя или пространство имен типа XML
```

Пример использования XML сериализации:
```csharp
[Serializable]
public class Person
{
    public string Name { get; set; }
    [XmlAttribute]
    public int Age { get; set; } //дополнительно кодирование как атрибута
    [NonSerialized] 
    public string Secret = "XF-1244";
    public Person() {} //стандартный конструктор для xml сериализации
}
[Serializable]
public class Home
{
    public Person Person { get; set; }
    public int Size { get; set; }
    public Home() {} //стандартный конструктор для xml сериализации
}
[Serializable, XmlRoot(Namespace = "http://www.mycompany.ru")] //специальное уточнение пространства имен XML
public class BigHome : Home
{
    public int Count { get; set; }
    public BigHome() {} //стандартный конструктор для xml сериализации
}
class Program
{
    static void Main()
    {
        SaveXml();
        ReadNPrintXml();
        Console.WriteLine("Нажмите любую кнопку для выхода...");
        Console.ReadKey();
        void SaveXml()
        {
            Person person = new Person
            {
                Name = "Тестовое имя",
                Age = 22,
            };
            BigHome home = new BigHome
            {
                Person = person,
                Size = 10,
                Count = 1,
            };
            XmlSerializer xmlSerializer = new XmlSerializer(typeof(BigHome));
            using (FileStream stream = new FileStream("data.xml",FileMode.Create,FileAccess.Write,FileShare.None))
            {
                xmlSerializer.Serialize(stream, home);
            }
            Console.WriteLine("Объект сохранен в Xml формате...");
        }
        void ReadNPrintXml()
        {
            XmlSerializer xmlSerializer = new XmlSerializer(typeof(BigHome));
            using (FileStream stream = File.OpenRead("data.xml"))
            {
                BigHome home = (BigHome)xmlSerializer.Deserialize(stream);
                Console.WriteLine($"Прочитанный объект: {home.Person.Name}, {home.Person.Age}, {home.Size}, {home.Count}");
            }
        }
    }
}
```
Пример сериализации коллекции объектов:
```csharp
//теже объекты что и выше
class Program
{
    static void Main()
    {
        SaveListToXml();
        ReadNPrintListFromXml();
        Console.WriteLine("Нажмите любую кнопку для выхода...");
        Console.ReadKey();
        void SaveListToXml()
        {
            List<BigHome> myHomes = new List<BigHome>();
            myHomes.Add(new BigHome
            {
                Person = new Person { Name = "Тестовое имя", Age = 22, }, Size = 10, Count = 1,
            });
            myHomes.Add(new BigHome
            {
                Person = new Person { Name = "Дополнительное имя", Age = 18, }, Size = 15, Count = 3,
            });
            myHomes.Add(new BigHome
            {
                Person = new Person { Name = "Третье имя", Age = 32, }, Size = 30, Count = 2,
            });
            XmlSerializer xmlSerializer = new XmlSerializer(typeof(List<BigHome>));
            using (FileStream stream = new FileStream("list.xml",FileMode.Create,FileAccess.Write,FileShare.None))
            {
                xmlSerializer.Serialize(stream, myHomes);
            }
            Console.WriteLine("Объект сохранен в Xml формате...");
        }
        void ReadNPrintListFromXml()
        {
            XmlSerializer xmlSerializer = new XmlSerializer(typeof(List<BigHome>));
            using (FileStream stream = new FileStream("list.xml",FileMode.Open,FileAccess.Read,FileShare.None))
            {
                List<BigHome> myHomes = xmlSerializer.Deserialize(stream) as List<BigHome>;
                Console.WriteLine("Прочитанный список:");
                foreach (var m in myHomes)
                    Console.WriteLine($"Прочитанный объект: {m.Person.Name}, {m.Person.Age}, {m.Size}, {m.Count}");
            }
        }
    }
}
```

XML очень похож на HTML. Он очень удобен для хранения информации, которая должна переноситься с компьютера на компьютер. Он может применятся для хранения информации проектов. Но XML был создан для описания данных с прицелом на то, что представляют собой данные. HTML был создан для отображения данных с прицелом на то, как выглядят отображаемые данные.

XML расшифровывается как «расширяемый язык разметки» (EXtensible Markup Language). XML — язык разметки, похожий на HTML. XML был создан для описания данных. Теги XML не предопределены. Вы можете использовать свои теги.

Простой пример XML-файла:
```csharp
<?xml version="1.0"?>
<ArrayOfSwine xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema">
  <Swines>
    <name>Свинка - бринка</name>
    <age>10</age>
  </Swines>
</ArrayOfSwine>
```

Применяя технологию сериализации, очень просто сохранять большие объемы данных с минимальными усилиями. Давайте рассмотрим этот процесс на примере сериализации созданного нами класса в формат XML-файла. Класс, объекты которого подлежат сериализации и десериализации, снабжается атрибутом Serializable. Этот класс должен быть публичным, иначе методы, которые реализуют работу по сериализации и десериализации, не смогут получить к нему доступ.

У класса должен быть установлен модификатор [Serializsble]. Поля, которые нужно сериализовать должны быть публичными. Если поле публичное, для него должно быть тогда публичное свойство. Конструктор без параметров создается автоматически для сериализуемого класса.

У поля можно ставить атрибут [NonSerialized], чтобы это поле не сериализовалось.

Простой пример класса:
```csharp
[Serializable]
public class Person
{
    public string Name;
    public string SurName;
    public int Age;
}
```
Простой пример с применением одного экземпляра этого класса:
```csharp
class Program
{
    static void SavePersonToXML(Person obj, string fileName)
    {
        XmlSerializer xmlFormat = new XmlSerializer(typeof(Person)); //формат сериализации
        FileStream fs = new FileStream(fileName, FileMode.Create, FileAccess.Write); //файловый поток
        xmlFormat.Serialize(fs, obj); //сериализация
        fs.Close();
    }
    static Person LoadFromXMLToPerson(string fileName)
    {
        Person obj = new Person();
        XmlSerializer xmlFormat = new XmlSerializer(typeof(Person)); //формат сериализации
        FileStream fs = new FileStream(fileName, FileMode.Open, FileAccess.Read); //файловый поток
        obj = xmlFormat.Deserialize(fs) as Person; //десериализация
        fs.Close(); 
        return obj;
    }
    static void Main()
    {
        Person person = new Person() {Name = "Вася", SurName = "Пупкин", Age = 23};
        SavePersonToXML(person, @"..\..\data.xml");
        Person person2 = LoadFromXMLToPerson(@"..\..\data.xml");
        Console.WriteLine($"Десериализованный: {person2.Name} {person2.SurName} {person2.Age} лет");
        Console.ReadKey();
    }
}
```
Простой пример с применением коллекции из экземпляров этого класса:
```csharp
class Program
{
    static void SaveListPersonToXML(List<Person> obj, string fileName)
    {
        XmlSerializer xmlFormat = new XmlSerializer(typeof(List<Person>)); //формат сериализации
        FileStream fs = new FileStream(fileName, FileMode.Create, FileAccess.Write); //файловый поток
        xmlFormat.Serialize(fs, obj); //сериализация
        fs.Close();
    }
    static List<Person> LoadXMLToListPerson(string fileName)
    {
        List<Person> obj = new List<Person>();
        XmlSerializer xmlFormat = new XmlSerializer(typeof(List<Person>)); //формат сериализации
        FileStream fs = new FileStream(fileName, FileMode.Open, FileAccess.Read); //файловый поток
        obj = xmlFormat.Deserialize(fs) as List<Person>; //десериализация
        fs.Close(); 
        return obj;
    }
    static void Main()
    {
        List<Person> list = new List<Person>
        {
            new Person { Name = "Вася", SurName = "Петров", Age = 18}, 
            new Person { Name = "Тимофей", SurName = "Иванов", Age = 20}, 
            new Person { Name = "Николай", SurName = "Кузьков", Age = 24}, 
        };
        SaveListPersonToXML(list, @"..\..\data.xml");
        List<Person> list2 = LoadXMLToListPerson(@"..\..\data.xml");
        foreach (var item in list2)
        {
            Console.WriteLine($"Десериализованный: {item.Name} {item.SurName} {item.Age} лет");
        }
        Console.ReadKey();
    }
}
```
Пример класса который будет более компактно сериализоватся т.к. поля будут в атрибутах:
```csharp
[Serializable]
public class Person
{
    [XmlAttribute]   
    public string Name;
    [XmlAttribute]   
    public string SurName;
    [XmlAttribute]   
    public int Age;
}
```

## Настройка процесса сериализации

Некоторые типы пространства имен Serialization для настройки:
```csharp
Член                    Описание
[ISerializable]         Интерфейс может быть реализован типом [Serializable] для управления его сериализацией и десериализацией
ObjectIDGenerator       Генерирует индетификаторы для членов в графе объектов
[OnDeserialized]        Метод, который будет вызван немедленно после десериализации объекта
[OnDeserializing]       Метод, который будет вызван перед десериализацией
[OnSerialized]          Метод, который будет вызван после сериализации
[OnSerializing]         Метод, который будет вызван перед началом сериализации
[OptionalField]         Определение поле типа, которое может быть пропущено в потоке
SerializationInfo       Пакет свойств "имя-значение" о состоянии объекта во время процесса сериализации
```

Пример настройки процесса сериализации интерфейсом ISerializable:
```csharp
[Serializable]
class MyData : ISerializable
{
    private string name = "имя";
    private double money = 1_000_000.0;
    public MyData(){}
    protected MyData(SerializationInfo info, StreamingContext context) //конструктор для десериализации
    {
        name = info.GetString("Name").ToLower(); //восстановление членов из потока
        money = info.GetDouble("Money") * 1_000.0; //восстановление зарплаты
    }
    public void GetObjectData(SerializationInfo info, StreamingContext context)
    {
        info.AddValue("Name", name.ToUpper()); //наполнение данными потока
        info.AddValue("Money",money / 1_000); //сокрытие зарплата
    }
    public (string name, double money) Deconstruct() => (name, money);
}
class Program
{
    static void Main()
    {
        MyData myData = new MyData();
        SoapFormatter soapFormatter = new SoapFormatter();
        using (FileStream fs = new FileStream("mydata.soap",FileMode.Create,FileAccess.Write,FileShare.None))
        {
            soapFormatter.Serialize(fs, myData);
        }
        using (FileStream fs = File.OpenRead("mydata.soap"))
        {
            myData = soapFormatter.Deserialize(fs) as MyData;
            var (n, m) = myData.Deconstruct();
            Console.WriteLine($"Прочитанные данные: {n} {m}");
        }
        Console.WriteLine("Нажмите любую кнопку для выхода...");
        Console.ReadKey();
    }
}
```

Пример настройки процесса сериализации с помощью атрибутов:
```csharp
[Serializable]
class MyData
{
    private string name = "имя";
    private double money = 1_000_000.0;
    public MyData(){}
    [OnSerializing]
    private void OnSerializing(StreamingContext context)
    {
        name = name.ToUpper();
        money = money / 1_000.0;
    }
    [OnDeserialized]
    private void OnDeserialized(StreamingContext context)
    {
        name = name.ToUpper();
        money = money * 1_000.0;
    }
    public (string name, double money) Deconstruct() => (name, money);
}
```

# JSON сериализация

Пример сериализации в json формат файла:
```csharp
System.IO.File.WriteAllText("data.json", JsonConvert.SerializeObject(workers));
List<Worker> list = JsonConvert.DeserializeObject<List<Worker>>(System.IO.File.ReadAllText("data.json"));
```



