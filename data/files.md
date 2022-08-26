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
string[] drives = Directory.GetLogicalDrives();
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
















