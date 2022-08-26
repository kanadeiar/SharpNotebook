# Файлы

Пространство имен System.IO содержит классы службы файлового ввода. Еще содержит классы ввода-вывода в памяти.

Directory & File - эти два типа реализуют свои возможности с помощью статических методов, коэтом данные классы можно использовать без создания соответствующих им объектов.

DirectoryInfo & FileInfo обладают сходижим возможностями, но требуют создания соответствующих им объектов.

Абстрактный класс FileSystemInfo дает основные свойства для работы с файловой системой этим производным от него классам.

## Работа с директориями

Класс Directorylnfo позволяет работать с папками файловой системы.

```csharp
var dir1 = new DirectoryInfo("."); //текущий каталог
var dir2 = new DirectoryInfo(@"C:\Temp"); //каталог хлама
dir2.Create(); //создание новой папки с хламом, т.к. он отсутствует
```

Если разрабатываются приложения .NET Core для разных платформ, тогда должны применять конструкции Path.VolumeSeparatorChar и Path.DirectorySeparatorChar, которые будут выдавать подходящие символы на основе платформы

```csharp
var dir3 = new Directorylnfo($@"С{Path.VolumeSeparatorChar}{Path.DirectorySeparatorChar}MyCode{Path.DirectorySeparatorChar}Testing");
```

Пример работы с папками и дисками:

```csharp
var dir = new DirectorуInfо(@"C:\Windows\Web\Wallpaper");
var imageFiles = dir.GetFiles("*.jpg", SearchOption.AllDirectories);
Console.WriteLine("Найдено {0} *.jpg файлов\n", imageFiles.Length);
dir = new DirectoryInfo("."); //место установки приложения
dir.CreateSubdirectory("МояПапка");
var myDir = dir.CreateSubdirectory(@"МояПапка\Данные");
Console.WriteLine($"Новая папка: {myDir}");
Console.WriteLine("Нажмите любую кнопку для продолжения");
Console.ReadKey();
var drives = Directory.GetLogicalDrives();
Console.WriteLine("Названия дисков:");
foreach (var d in drives)
    Console.WriteLine($"- {d}");
var myDrives = DriveInfo.GetDrives();
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
```

## Работа с файлами

Класс Filelnfo предназначен для организации доступа к физическому файлу, который содержится на жестком диске компьютера. Он позволяет получать информацию об этом файле (например, о времени его создания, размере, атрибутах и т.п.), а также производить различные операции, например, по созданию файла или его удалению.

Метод Create() возвращает файловый поток, который предоставляет синхронную и асинхронную операции записи/чтения лежащего в его основе файла. Открывает полный доступ по чтению и записи всем пользователям.

```csharp
var filename = $@"c{Path.VolumeSeparatorChar}{Path.DirectorySeparatorChar}temp{Path.DirectorySeparatorChar}test.dat";
var f = new FileInfo(filename);
using (var fs = f.Create())
```

С помощью метода Open() можно открывать существующие файлы и создавать новые файлы с более высокой точностью представления.

```csharp
using (var fs = f.Open(FileMode.OpenOrCreate, FileAccess.ReadWrite, FileShare.None))
```

Методы OpenRead() и OpenWrite() позволяют получить в работу файл в только для чтения или только для записи без указания параметров. 

Метод OpenText() вертает уже объект типа StreamReader, позволяющий читать символьные данные. Метод CreateText() и AppendText() вертают объект типа StreamWriter, позволяющий записывать символьные данные.

Тип File позволяет с помощью определенных в нем статических методов работать с файлами.

Пример работы с файлом:

```csharp
var filename = $@"c{Path.VolumeSeparatorChar}{Path.DirectorySeparatorChar}temp{Path.DirectorySeparatorChar}test.dat";
using (var fs = File.Create(filename))
{ }
File.Delete(filename);
using (var fso = File.Open(filename, FileMode.OpenOrCreate, FileAccess.ReadWrite, FileShare.None))
{ }
using (var rofs = File.OpenRead(filename))
{ }
using (var wofs = File.OpenWrite(filename))
{ }
using (var sr = File.OpenText(filename))
{ }
using (var sw = File.CreateText(filename))
{ }
using (var sa = File.AppendText(filename))
{ }
```

В классе определены дополнительные методы для удобства, позволяеющие в одну инструкцию открыть файл, произвести необходимые операции и закрыть файл: 

- ReadAllBytes(), 

- ReadAllLines(), 

- ReadAllText(), 

- WriteAllBytes(), 

- WriteAllLines(), 

- WriteAllText().

```csharp
var names = new string[] { "Иван", "Петр", "Сидор", "Вася", "Коля", "Дима" };
File.WriteAllLines("test.txt", names);
foreach (var e in File.ReadAllLines("test.txt"))
{
    Console.WriteLine(e);
}
```

## Абстрактный поток

Абстрактный класс Stream в мире манипуляций вводом-выводом представляет порцию данных, протекующую между источником и приемником. Потоки предоставляют общий способ взаимодействия с последовательностью байтов независимо от устройства, вида хранения и отображения байтов.

Потомки этого класса представляют данные в виде низкоуровневых байтов. Методы этого абстрактного класса позволяют манипулировать конктектным потоком байтов.

Close() - закрывает поток и освобождает ресурсы

Flush() - обновляет источник данных или буфер, затем очищает буфер.

Length - длинна потока в байтах

Position - определяет текущую позицию в потоке

Read(), ReadByte(), ReadAsync() - читают последовательность байтов из потока и перемещает позицию потока вперед

Seek() - устанавливает позицию в текущем потоке

SetLength() - устанавливает длинну текущего потока

Write(), WriteByte(), WriteAsync() - запись последовательности байтов в текущий поток и перемещает текущую позицию вперед

## Файловый поток

Класс FileStream представляет детальную реализацию потока для потоковой работы с файлами. Может оперировать только с низкоуровневыми байтами. Объекты нужно кодировать в байтовый массив.

Пример работы с файловым потоком:

```csharp
using (var fs = File.Open("test.dat", FileMode.Create))
{
    var message = "Привет";
    var writeBytes = Encoding.Default.GetBytes(message);
    fs.Write(writeBytes, 0, writeBytes.Length);
    fs.Position = 0;
    var readBytes = new byte[fs.Length];
    for (var i = 0; i < fs.Length; i++)
    {
        readBytes[i] = (byte)fs.ReadByte();
        Console.WriteLine(readBytes[i]);
    }
    Console.Write("Сообщение:");
    Console.WriteLine(Encoding.Default.GetString(readBytes));
}
```

Классы StreamWriter и StreamReader позволяют читать и записывать символьные данные. По умолчанию работают с символами Unicode, но можно это изменить.

Пример записи текста в файл:

```csharp
using (var writer = File.CreateText("test.txt"))
{
    writer.WriteLine("Test text");
}
```

Пример чтения этого файла:

```csharp
using (var reader = File.OpenText("test.txt"))
{
    var buffer = default(string);
    while ((buffer = reader.ReadLine()) != null)
    {
        Console.WriteLine(buffer);
    }
}
```

## Поток символов в памяти

Классы StringWriter и StringReader можно использовать для трактовки информации как потока символов в памяти. 

Пример работы с символьными потоками:

```csharp
using (var swriter = new StringWriter())
{
    swriter.WriteLine("Test");
    var sb = swriter.GetStringBuilder();
    sb.Insert(0, "Hey!!! ");
    Console.WriteLine("Copy: {0}", swriter);
    using (var sreader = new StringReader(swriter.ToString()))
    {
        var input = default(string);
        while ((input = sreader.ReadLine()) != null)
        {
            Console.WriteLine(input);
        }
    }
}
```

## Поток в двоичном формате

Классы BinaryWriter и BinaryReader позволяют читать и записывать в поток дискретные типы данных в компактном двоичном формате. 

Пример:

```csharp
var f = new FileInfo("test.dat");
using (var bw = new BinaryWriter(f.OpenWrite()))
{
    System.Console.WriteLine(bw.BaseStream);
    var a = 1234.5;
    var i = 12345;
    var s = "1,2,3";
    bw.Write(a);
    bw.Write(i);
    bw.Write(s);
}
using (var br = new BinaryReader(f.OpenRead()))
{
    Console.WriteLine(br.ReadDouble());
    Console.WriteLine(br.ReadInt32());
    Console.WriteLine(br.ReadString());
}
```

## Буферизированный поток

Самым быстрым способом чтения и записи данных является буферизированный поток.

```csharp
using (var fs = new FileStream("data.bin", FileMode.Create, FileAccess.Write))
{
    var countPart = 4;
    var bufSize = (int) (8 / countPart);
    var buffer = new byte[8];
    for (var i = 0; i < buffer.Length; i++)
    {
        buffer[i] = (byte)i;
    }
    using (var bs = new BufferedStream(fs, bufSize))
    {
        for (var i = 0; i < countPart; i++)
        {
            bs.Write(buffer, i * bufSize, bufSize);
        }
    }
}
using (var fs = new FileStream("data.bin", FileMode.Open, FileAccess.Read))
{
    var countPart = 4;
    var bufSize = (int)fs.Length / countPart;
    var arr = new byte[fs.Length];
    var reader = new BufferedStream(fs);
    for (var i = 0; i < countPart; i++)
    {
        reader.Read(arr, i * bufSize, bufSize);
    }
    foreach (var el in arr)
    {
        Write(el + " ");
    }
}
```

## Слежение за файлами

Класс FileSystemWatcher нужен для программного отслеживания файлов на предмет любых действий, которые содержатся в перечислении NotifyFilters - Attributes, CreationTime, FirectoryName, FileName, LastAccess, LastWrite, Security, Size.

Пример слежения за файлом:
```csharp
static void Main()
{
    var watcher = new FileSystemWatcher();
    try
    {
        watcher.Path = @".";
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





