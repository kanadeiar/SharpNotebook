# Библиотеки классов

## Сборки

Приложения .NET создаются путем соединения в одно целое множества сборок. Содействуют многократному использованию кода. Библиотека кода может быть как *.dll, так и *.exe, достигая возможности многократного использования кода, нейтрально к языку.

Сборки определяют свои уникальные имена, и содержащиеся в них классы с одинаковыми именами будут рассматриватся как уникальные.

Если номер сборки не указан, то автоматом назначается версия 1.0.0.0. Использование номера позволяет множеству версий одно й и той же самой сборки существовать на одной машине.

Каждая сборка самоописательная, содержит информацию о всех внешних сборках, которые нужны для нормальной работы. Каждая сборка содержит метаданные, описывающие содержащиеся в сборке каждого типа.

Закрытые сборки размещаются в том же каталоге, что и клиенское приложение. Разделяемые сборки - для потребления многочисленными приложениями на одиночной машине, развертываются в специальном каталоге, называемом глобальным кешем сборок (Global Assembly Cache - GAC). Приложения с помощью конфигфайлов XML можно настроивать на поиск зависимых сборок в определенных местоположениях.

## Состав сборок

Внутри сборка состоит из нескольких элементов.

Заголовок файла устанавливает то, в каких средах можно ее загружать, описывает тип приложения - консолькое, графическое, библиотека. 

Заголовок файла CLR блок данных необходимых для его понимания средой CLR. 

Код CIL, промежуточной язык, не зависящий от платформы и процессора, компилируемый на лету JIT-компилятором в инструкции, специфичные для платформы выполнения. Позволяет выполнять сборку на разнообрахных архитектурах, устройствах и операционных системах.

Метаданные, полностью описывающие формат внутренних типов и внешние типы, на которые сборка ссылается. Необходимы для размещения типов в памяти и для упрощения удаленного вызова методов.

Манифест, документирующий каждый модуль внутри сборки, устанавливающий версию этой сборки и внешние сборки, на которые ссылается текущая сборка. Нужен для нахождения ссылок на внешние сборки.

Дополнительные ресурсы сборки. Сборка может содержать любое количество встроенных ресурсов, таких как файлы изображения, звуковые клипы, таблицы строк. Платформа поддерживает подчиненные сборки, содержащие только локализованные ресурсы и больше ничего.

## Разрешение конфликтов псевдонимами

Используя ключевое слово using можно создавать псевдонимы для полностью заданного типа.

Пример:
```csharp
using SamForm = System.Runtime.Serialization.Formatters.Binary;
namespace ConsoleApp1
{
    class Program
    {
        public static void Main()
        {
            SamForm.BinaryFormatter s = new SamForm.BinaryFormatter();
            Console.ReadLine();
...
```

## Простая библиотека, закрытая сборка

Проект "Библиотека классов (.NET Framework)"

Добавить ссылку "System.Windows.Forms"

Файл "Person.cs":
```csharp
namespace ClassLibrary1
{
    public enum State //текущее состояние работника
    {
        Work,
        Free,
    }
    public abstract class Person
    {
        public string Name { get; set; }
        public int Age { get; set; }
        public int Salary { get; set; }
        protected State state = State.Free;
        public State CurrState => state;
        public abstract void AddSalary();
        public Person() { }
        public Person(string name, int age, int salary)
        {
            Name = name;
            Age = age;
            Salary = salary;
...
```
Файл "Persons.cs":
```csharp
using System.Windows.Forms;
namespace ClassLibrary1
{
    public class Worker : Person //работник
    {
        public Worker() { }
        public Worker(string name, int age, int salary) 
            : base(name, age, salary) { }
        public override void AddSalary() //прибавка зарплаты
        {
            Salary += 100;
            MessageBox.Show($"Получение зарплаты {Salary}");
        }
    }
    public class Manager : Person //менеджер
    {
        public Manager() { }
        public Manager(string name, int age, int salary) 
            : base(name, age, salary) { }
        public override void AddSalary() //прибавка зарплаты
        {
            Salary += 10000;
            MessageBox.Show($"Получение бонуса {Salary}!");
...
```

Использование этой библиотеки:

Добавить ссылку на "ClassLibrary1".
```csharp
using ClassLibrary1;
namespace ConsoleApp1
{
    class Program
    {
        public static void Main()
        {
            Worker worker = new Worker("Вася",18,5000);
            worker.AddSalary();
            Manager manager = new Manager("Петя",20,10000);
            manager.AddSalary();
            Console.WriteLine("Нажмите любую кнопку");
            Console.ReadLine();
...
```

Конфигурационный файл настройки закрытых сборок. Отредактированный файл "App.config" копируется в каталог "Debug", при этом изменяя имя на название исполняемого файла. 

Файл "App.config":
```csharp
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
    <startup> 
        <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.7.1" />
    </startup>
    <runtime>
        <assemblyBinding xmlns="urn:schemas-microsoft-com:asm.v1">
          <probing privatePath="MyLibraries;MyLibraries\Test"/>
        </assemblyBinding>
    </runtime>
</configuration>
```

С таким конфигфайлом можно копировать файлы dll в папку "MyLibraries", при запуске приложения среда CLR будет зондировать файл dll в этой новой папке.

## Разделенная сборка

До версии .NET 4.0 глобальный кеш сборок размещался в "C:\Windows\assembly", там все что 1.0, 2.0, 3.0, 3.5.

С версии .NET 4.0 глобальный кэш сборок - библиотеки разделяемых сборок размещаются в каталоге "C:\Windows\Microsoft.NET\assembly\GAC_MSIL". В этом месте библиотеки размещаются в виде имени отдельной библиотеки кода. Внутри еще один подкаталог именуемый по соглашению "v4.0_старшийНомер.младшийНомер.номерСборки.номерРеакции_значМаркераОткрытогоКлюча".

Устанавливать туда можно только сборки с рашришением dll.

Строгое имя сборки обеспечивает уровень защиты от потенциальной поддерки злыднями.

Строгое имя состоит из набора связанных данных:

- дружественное имя сборки

- номер версии сборки

- значение открытого ключа

- необязательное значение, показывающее культуру

- встроенная цифровая подпись, созданная с использованием хэш-кода содержимого сборки и значения секретного ключа.

Генерация строних имен:

1. "Проект"->"Свойства"->флаг"Подписать сборку"->"Создать..."

2. Ввести имя и установить пароль.

Теперь при каждой компиляции файл будет подписываться.

Установка сборка библиотеки в GAC помимо установочного пакета возможно использванием инстумента командной строки gacutil.exe.
```csharp
/i устанавливает сборку со строгим именем в GAC
/u удаляет сборку из GAC
/l отображает список сборок в GAC
```

Пример выполения команд:
```csharp
//установка библиотека в GAC
C:\Program Files (x86)\Microsoft SDKs\Windows\v10.0A\bin\NETFX 4.8 Tools>gacutil.exe /i D:\Develop\cppGB\CppConsoleApplications\ClassLibrary1\bin\Debug\ClassLibrary1.dll
//проверка что она там есть
C:\Program Files (x86)\Microsoft SDKs\Windows\v10.0A\bin\NETFX 4.8 Tools>gacutil.exe /l ClassLibrary1
```

Потребление разделяемой сборки:

"Добавить ссылку"->файл dll проекта

Установить свойство ссылки "Копировать локально" в False

Убедиться что рядом с файлом exe отустствует dll.

Теперь при добавлении ссылки на сборку, манифест которой содержит значение publickey, среда VisualStudio считает, что такая строго именованная сборка будет развернута в GAC, и поэтому не копирует ее двоичный файл.

## Обновление разделяемой сборки

Пример обновления библиотеки:
Файл "Persons.cs":
```csharp
using System.Windows.Forms;
namespace ClassLibrary1
{
    public class Worker : Person //работник
    {
        public Worker() { }
        public Worker(string name, int age, int salary) 
            : base(name, age, salary) { }
        public override void AddSalary() //прибавка зарплаты
        {
            Salary += 100;
            MessageBox.Show($"Версия сборки 2.0!\nПолучение зарплаты {Salary}"); //обновление
        }
    }
...
```
Необходимо изменить номер версии с 1.0.0.0 на 2.0.0.0 в окне "Версия сборки":

Далее выполнить туже команду установки библиотеки в GAC:
```csharp
//установка библиотека в GAC
C:\Program Files (x86)\Microsoft SDKs\Windows\v10.0A\bin\NETFX 4.8 Tools>gacutil.exe /i D:\Develop\cppGB\CppConsoleApplications\ClassLibrary1\bin\Debug\ClassLibrary1.dll
//проверка что теперь там две версии
C:\Program Files (x86)\Microsoft SDKs\Windows\v10.0A\bin\NETFX 4.8 Tools>gacutil.exe /l ClassLibrary1
```

Теперь требуется перенаправить сборку на специфическую версию разделяемой сборки. Файл "App.config":
```csharp
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
    <startup> 
        <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.7.1" />
    </startup>
    <runtime>
        <assemblyBinding xmlns="urn:schemas-microsoft-com:asm.v1">
          <assemblyIdentity name="ClassLibrary1"
                            publicKeyToken="2EBFFCE843C773FC"
                            culture="neutral"/>
          <bindingRedirect oldVersion="1.0.0.0"
                           newVersion="2.0.0.0"/>
        </assemblyBinding>
    </runtime>
</configuration>
```

## Кодовые базы

В конфиг файлах допустимо указывать кодовые базы с помощью <codeBase>, позволяющий инструктировать среду CLR о необходимости зондирования зависимых сборок, находящихся в произвольных местоположениях на машине за пределами клиенского каталога или в интернете.
    
Сборка будет загружатся по требованию в специальный каталог внутри GAC, который называется кэш загрузок.

Пример со ссылкой на файл на другой машине:
```csharp
<configuration>
...
    <runtime>
        <assemblyBinding xmlns="urn:schemas-microsoft-com:asm.v1">
          <assemblyIdentity name="ClassLibrary1"
                            publicKeyToken="2EBFFCE843C773FC"
                            culture="neutral"/>
          <bindingRedirect oldVersion="2.0.0.0"
                            href="file:///C:/MyApp/ClassLibrary1.dll"/>
        </assemblyBinding>
    </runtime>
</configuration>
```

Пример со ссылкой на файл в интернете:
```csharp
<configuration>
...
    <runtime>
        <assemblyBinding xmlns="urn:schemas-microsoft-com:asm.v1">
          <assemblyIdentity name="ClassLibrary1"
                            publicKeyToken="2EBFFCE843C773FC"
                            culture="neutral"/>
          <bindingRedirect oldVersion="2.0.0.0"
                            href="http://www.MySite.ru/Assemblies/ClassLibrary1.dll"/>
        </assemblyBinding>
    </runtime>
</configuration>
```


