# Файловая система (Директории, файлы, слежение)

Пространство имен System.IO содержит классы службы файлового ввода и вывода и ввода и вывода в памяти.

Некоторые члены этого пространства:
```csharp
BinaryReader,BinaryWriter   Сохранение и извлечение даннных элементарных типов как двоичные значения
BufferedDtream              Временное хранилище для потока байтов, который может быть передан куда то позже
Directory,DirectoryInfo     Классы применяются для манипулирования структурой каталогов машины. 
DriveInfo                   Класс предоставляет детальную информацию о дисковых устройствах на машине
File, FileInfo              Классы для манипулирования набором файлов на машине. 
FileStream                  Класс произвольного доступа к файлу в виде потока байтов
FileSystemWatcher           Класс отслеживания модификации внешних файлов в указанном каталоге
MemoryStream                Класс произвольного доступа к данным, хранящимся в памяти, а не на физ носителе
Path                        Выполнение операций на String, где находится информация о пути к файлу или каталогу, в независимой от платформы манере
StreamWriter,StreamReader   Классы хранения текстовой информации в файле без поддержки произвольного доступа к файлу
StringWriter,StringReader   Классы работы с текстовой информацией в файле на основе строкового буфера
```

В языке C# предусмотрены следующие классы для работы с файловой системой комьютера:

Directory & File - эти два типа реализуют свои возможности с помощью статических методов, коэтом данные классы можно использовать без создания соответствующих им объектов.

DirectoryInfo & FileInfo обладают сходижим возможностями, но требуют создания соответствующих им объектов.

Некоторые члены класса FileSystemInfo
```csharp
Attributes                  Получение или утановка аттрибутов файла, представленных перечислением FileAttributes
CreationTime                Время создания текущего файла или каталога
Exists                      Существует ли данный файл или каталог
Extension                   Извлекает расширение файла
FullName                    Полный путь к файлу/каталогу
LastAccessTime              Время последнего доступа к текущему файлу/каталгу
LastWriteTime               Время последней записи к текущему файлу/катлогу
Name                        Имя текущего файла или каталога
```

## Работа с директориями

Некоторые методы класса DirectoryInfo
```csharp
Метод                       Описание
Create()                    Создают каталог (или подкаталог) по указанному
CreateSubDirectory()        пути в файловой системе
Delete()                    Удаляет пустой каталог
GetDirectories()            Позволяет получить доступ к подкаталогам текущего каталога (в виде массива объектов DirectoryInfo)
GetFiles()                  Позволяет получить доступ к файлам текущего каталога (в виде массива объектов FileInfo)
MoveTo()                    Перемещает каталог и все его содержимое на новый адрес в файловой системе
Parent                      Дает родительский каталог данного каталога
Root                        Получает корневую часть пути
```
Работа с типом DirectoryInfo начинается с создания экземпляра класса (объект):
```csharp
DirectoryInfo dir1 = new DirectoryInfo("."); //текущий каталог
DirectoryInfo dir2 = new DirectoryInfo(@"C:\Temp"); //каталог хлама
dir2.Create(); //создание новой папки с хламом, т.к. он отсутствует
```
Пример получения информации о каталоге:
```csharp
static void Main() 
{
    DirectoryInfo dir = new DirectoryInfo(@"c:\Windows");
    Console.WriteLine($"Полное имя: {dir.FullName}");
    Console.WriteLine($"Имя каталога: {dir.Name}");
    Console.WriteLine($"Родительский: {dir.Parent}");
    Console.WriteLine($"Время создания: {dir.CreationTime}");
    Console.WriteLine($"Атрибуты: {dir.Attributes}");
    Console.WriteLine("Нажмите любую кнопку ...");
    Console.ReadKey();
}
```
Пример работы с папками:
```csharp
static void Main() 
{
    DirectoryInfo dir = new DirectoryInfo(@"d:\");
    dir.CreateSubdirectory("МояПапка");
    dir.CreateSubdirectory(@"МояПапка\Тест");
    dir = new DirectoryInfo("."); //место установки приложения
    dir.CreateSubdirectory("МояПапка");
    DirectoryInfo myDir = dir.CreateSubdirectory(@"МояПапка\Данные");
    Console.WriteLine($"Новая папка: {myDir}");
    Console.WriteLine("Нажмите любую кнопку для продолжения");
    Console.ReadKey();
    string[] drives = Directory.GetLogicalDrives();
    Console.WriteLine("Названия дисков:");
    foreach (var d in drives)
        Console.WriteLine($"- {d}");
    DriveInfo[] myDrives = DriveInfo.GetDrives();
    foreach (var d in myDrives)
    {
        Console.WriteLine($"Название: {d.Name} и тип: {d.DriveType}");
        if (d.IsReady)
        {
            Console.WriteLine($"Свободное место: {d.TotalFreeSpace} байт");
            Console.WriteLine($"Формат: {d.DriveFormat}");
            Console.WriteLine($"Метка: {d.VolumeLabel}");
        }
    }
    try
    {
        Directory.Delete(@"d:\МояПапка",true); //true - удалить и внутренние каталоги
    }
    catch (IOException ex)
    {
        Console.WriteLine(ex.Message);
    }
    Console.WriteLine("Нажмите любую кнопку для завершения");
    Console.ReadKey();
}
```

## Работа с файлами

Класс Filelnfo предназначен для организации доступа к физическому файлу, который содержится на жестком диске компьютера. Он позволяет получать информацию об этом файле (например, о времени его создания, размере, атрибутах и т.п.), а также производить различные операции, например, по созданию файла или его удалению.

Некоторые свойства класса FileInfo
```csharp
Свойство                Описание
Exists                  Может быть использовано, чтобы определить, существует ли данный объект файловой системы
Attributes              Позволяет получить или установить атрибуты для данного объекта файловой системы. Для этого свойства используются значения и перечисления FileAttributes
FullName                Возвращает имя файла или каталога с указанием пути к нему в файловой системе
Name                    Возвращает имя указанного файла. Это свойство доступно только для чтения. Для каталогов возвращает имя последнего каталога в иерархии, если это возможно. Если нет, возвращает полностью определённое имя
```
Некоторые методы класса FileInfo
```csharp
Член                    Описание
AppendText()            Создание объекта StreamWriter и добавляет текст в файл
Create()                Создаёт новый файл и возвращает объект FileStream для взаимодействия с этим файлом
CreateText()            Создает объект StreamWriter, записывающий в новый текстовый файл
Delete()                Удаляет файл, которому соответствует объект FileInfo
Directory               Экземпляр родительского каталога
DirectoryName           Полный путь к родительскому каталогу
Length                  Возвращает размер файла
CopyTo()                Копирует уже существующий файл в новый 
MoveTo()                Перемещает файл в новое расположение, с указание нового имени файла
Name                    Получение имени файла
Open()                  Открывает файл с разнообразными привилегиями чтения/записи
OpenRead()              Создание объекта FileStream, только для чтения
OpenText()              Создание объекта StreamReader, для чтения из существующего файла
OpenWrite()             Создание объекта FileStream, только для записи
```
Пример получения информации о файлах в каталоге:
```csharp
static void Main() 
{
    DirectoryInfo dir = new DirectoryInfo(@"C:\Windows\Web\Wallpaper");
    FileInfo[] images = dir.GetFiles("*.jpg", SearchOption.AllDirectories);
    Console.WriteLine($"Найдено {images.Length} файлов");
    foreach (var i in images)
        Console.WriteLine($"Файл:{i.Name} Размер:{i.Length} Время создания:{i.CreationTime} Атрибуты:{i.Attributes}");
    Console.ReadKey();
}
```
Пример работы с файлами:
```csharp
static void Main() 
{
    FileInfo f = new FileInfo(@"d:\Test.dat");
    //FileStream fs = f.Create(); //создать новый файл
    //fs.Close();
    using (FileStream fs = f.Create()) //использование IDisposable
    { //использовать }
    //открыть или создать, чтение/запись, без совместного использования
    using (FileStream fs2 = f.Open(FileMode.OpenOrCreate,FileAccess.ReadWrite,FileShare.None))
    { }
    FileInfo f1 = new FileInfo(@"d:\Test.dat");
    using (FileStream readOnly = f1.OpenRead()) //открыть для чтения
    { }
    FileInfo f2 = new FileInfo(@"d:\Test.dat");
    using (FileStream writeOnly = f2.OpenWrite()) //открыть для записи
    { }
    FileInfo f3 = new FileInfo(@"d:\test.ini");
    using (StreamReader reader = f3.OpenText()) //открыть для чтения символьных данных
    { }
    FileInfo f4 = new FileInfo(@"d:\test.txt");
    using (StreamWriter writer = f4.CreateText()) //новый тектовый файл для записи
    { }
    FileInfo f5 = new FileInfo(@"d:\text.txt");
    using (StreamWriter writerAppend = f5.AppendText()) //добавление текста в файл
    { }
    Console.WriteLine("Нажмите любую кнопку для завершения");
    Console.ReadKey();
}
```

Класс File. В этом классе определено множество членов, идентичных функциональности FileInfo - AppendText(), Create(), CreateText(), Open(), OpenRead(), OpenWrite(), OpenText(). 

Есть дополнительные еще в классе File:
```csharp
Член                    Описание
ReadAllBytes()          Открывает файл, дает двоичные даннные массивом байт и закрывает файл
ReadAllLines()          Открывает файл, дает символьные данные массивом строк и закрывает файл
ReadAllText()           Открывает файл, дает символьные даные в виде String и закрывает
WriteAllBytes()         Открывает файл, записывает в него массив байт и закрывает файл
WriteAllLines()         Открывает файл, записывает в него массив строк и закрывает файл
WriteAllText()          Открывает файл, записывает в него символьные данные из строки и закрывает файл
```

Еще пример работы с файлами:
```csharp
static void Main() 
{
    using (FileStream fs1 = File.Create(@"d:\test.dat")) //создание файла
    { }
    //открыть для чтения
    using (FileStream fs3 = File.Open(@"d:\test.dat", FileMode.OpenOrCreate, FileAccess.ReadWrite, FileShare.None))
    { }
    using (FileStream readOnlyStream = File.OpenRead(@"d:\test.dat")) //чтение
    { }
    using (FileStream writeOnlyStream = File.OpenWrite(@"d:\test.dat")) //запись
    { }
    using (StreamReader sreader = File.OpenText(@"d:\test.ini")) //только для чтения
    { }
    using (StreamWriter swriter = File.CreateText(@"d:\test.txt")) //только для записи
    { }
    using (StreamWriter swriterAppend = File.AppendText(@"d:\test.txt")) //для добавления
    { }
    //запись и чтение данных из файла
    string[] myStrings = { "Тестовые данные", "Вызов чегонибудь", "Еще тестовые данные", "Дополнительная строка" };
    File.WriteAllLines(@"text.txt", myStrings); //запись всех данных
    foreach (var s in File.ReadAllLines(@"text.txt"))
        Console.WriteLine($"строка: {s}");
    Console.WriteLine("Нажмите любую кнопку для завершения");
    Console.ReadKey();
}
```

## Слежение за файлами

Класс FileSystemWatcher нужен для программного отслеживания файлов на предмет любых действий, которые содержатся в перечислении NotifyFilters - Attributes, CreationTime, FirectoryName, FileName, LastAccess, LastWrite, Security, Size.

Пример слежения за файлом:
```csharp
static void Main()
{
    FileSystemWatcher watcher = new FileSystemWatcher();
    try
    {
        watcher.Path = ".";
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.Message);
        return;
    }
    watcher.NotifyFilter = NotifyFilters.LastAccess |
                           NotifyFilters.LastWrite |
                           NotifyFilters.FileName |
                           NotifyFilters.DirectoryName;
    watcher.Filter = "*.txt"; //отслеживание только текстовых файлов
    watcher.Changed += new FileSystemEventHandler(OnChanged);
    watcher.Created += new FileSystemEventHandler(OnChanged);
    watcher.Deleted += new FileSystemEventHandler(OnChanged);
    watcher.Renamed += new RenamedEventHandler(OnRenamed);
    watcher.EnableRaisingEvents = true;
    Console.WriteLine("Нажмите кнопку 'q' для выхода");
    while (Console.Read() != 'q')
    { };
    Console.ReadKey();
}
static void OnChanged(object source, FileSystemEventArgs e)
{
    Console.WriteLine($"Файл: {e.FullPath} {e.ChangeType}");
}
static void OnRenamed(object source, RenamedEventArgs e)
{
    Console.WriteLine($"Файл {e.OldFullPath} изменен на {e.FullPath}");
}
```



