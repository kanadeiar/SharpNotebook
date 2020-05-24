## Сериализация

Термин «сериализация» описывает процесс сохранения состояния объекта в потоке (например, файловом потоке). Последовательность сохраняемых данных содержит всю информацию, необходимую для реконструкции (или десериализации) состояния объекта с целью последующего использования. 

Для сериализации можно использовать XML-файлы.

## XML-файлы
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

## Технология сериализации

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



