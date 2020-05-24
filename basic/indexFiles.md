# Файлы (потоки)

С#-программы выполняют операции ввода-вывода посредством потоков, которые построены на иерархии классов.

## Потоки

Центральную часть потоковой С#-системы занимает класс Stream пространства имен System.IO. Класс Stream представляет байтовый поток и является базовым для всех остальных потоковых классов. Основным для потоков является абстрактный класс System.IO.Stream. В нем определен ряд членов, обеспечивающих синхронное и асинхронное взаимодейтсие с хранилищем.

Из класса Stream выведены такие байтовые классы потоков, как:

FileStream — байтовый поток, разработанный для файлового ввода-вывода. 

BufferedStream заключает байтовый поток в оболочку и добавляет буферизацию, которая во многих случаях увеличивает производительность программы. 

MemoryStream — байтовый поток, который использует память для хранения данных.

Класс FileStream реализует абстрактный класс Stream, для потоковой работы с файлами, записывает или считывает только одиночный байт или массив байт. Еще существуют поток для памяти MemoryStream, поток для сетевого подключения NetworkStream.

Члены абстрактного класса Stream:
```csharp
Член                        Описание
CanRead,CanWrite,CanSeek    Определение, что поток поддерживает чтение, запись, поиск   
Close()                     Закрывает поток и освобождает ресурсы (сокеты, файловые дескрипторы), ассоциированные с потоком. Псевдоним Dispose()
Flush()                     Обновляет лежащий в основе источник/хранилище данных текущим состоянием буфера и очищает буфер
Length                      Дает длину потока в байтах
Position                    Текущая позиция в потоке
Read(),ReadByte(),ReadAsync()   Читает последовательность байтов из потока и перемещает текущую позицию потока вперед на кол-во прочитанных байтов
Seek()                      Установка позиции в потоке
SetLength()                 Установка длины потока
Write(),WriteByte(),WriteAsync()    Записывает последовательность байтов в текущий поток и перемещает позицию вперед на кол-во записанных байтов   
```

## Байтовый поток

Первонаперво создается объект класса FileStream.
```csharp
FileStream(string filename, FileMode mode, FileAccess how)
```
В нем указывается имя файла, режим открытия файла и способ доступа к файлу.

Для чтения очередного байта из потока, связанного с физическим файлом, используется метод ReadByte(). После прочтения очередного байта внутренний указатель перемещается на следующий байт файла. Если достигнут конец файла, метод ReadByte() возвращает значение -1.

Для побайтовой записи данных в поток используется метод WriteByte().

После работы файл необходимо закрыть.

Пример работы с файлом потоком FileStream:
```csharp
static void Main() 
{
    using (FileStream fstream = File.Open(@"d:\test.dat",FileMode.Create))
    {
        string msg = "Привет! Hello!"; //сообщение
        byte[] msgBytes = Encoding.Default.GetBytes(msg); //кодирование строки в виде масссива байтов
        fstream.Write(msgBytes,0,msgBytes.Length); //запись массива в файл
        fstream.Flush(); //вытолкнуть буфер в файл
        fstream.Position = 0; //позиция на начало
        Console.WriteLine("Сообщение в байтах:");
        byte[] bytesFromFile = new byte[msgBytes.Length];
        for (int i = 0; i < msgBytes.Length; i++)
        {
            bytesFromFile[i] = (byte) fstream.ReadByte();
            Console.Write(bytesFromFile[i]);
        }
        Console.WriteLine($"\nДекодированное: {Encoding.Default.GetString(bytesFromFile)}");
    }
    Console.WriteLine("Нажмите любую кнопку для завершения");
    Console.ReadKey();
}
```
Еще пример работы с файлом:
```csharp
//запись
using FileStream fileOut = new FileStream("data.txt", FileMode.Create, FileAccess.Write);
for (int i = 0; i < 5; i++)
{
    fileOut.WriteByte((byte)i); 
}
//чтение
using FileStream fileIn = new FileStream("data.txt", FileMode.Open, FileAccess.Read);
int j;
while ((j = fileIn.ReadByte())!=-1)
{
    Console.Write(j);
}
```

## Символьный поток

Класс StreamWriter является производным от абстрактного класса TextWriter, предоставляющий производным типам записывать текстовые данные в текущий символьный поток. Оболочка для класса FileStream;

Некоторые члены абстрактного класса TextWriter:
```csharp
Член                Описание
Close()             Метод закрывает средство записи и освобождает все свои ресурсы, тотже Dispode()
Flush()             Очищает все буферы средства записи и выталкивает все данные на лежащее в основе устройство, не закрывает средство записи
NewLine             Задает константу новой строки для класса средства записи.
Write()             Перегруженный метод записи данных в текстовый поток
WriteLine()         Перегруженный метод записи данных в текстовый поток с добавленрием новой строки
```
Пример записи в текстовый файл:
```csharp
static void Main() 
{
    //using (StreamWriter writer = File.CreateText("text.txt"))
    using (StreamWriter writer = new StreamWriter("text.txt"))
    {
        writer.WriteLine("Первая строка");
        writer.WriteLine("Вторая строка");
        writer.WriteLine("Номера:");
        for (int i = 0; i < 10; i++)
            writer.Write(i + " ");
        writer.Write(writer.NewLine); //новая строка
    }
    Console.WriteLine("Нажмите любую кнопку для завершения");
    Console.ReadKey();
}
```

Класс ReadWriter является производным от абстрактного класса TextReader, предоставляющий производным типам читать текстовые данные из текущего символьного потока. Оболочка для класса FileStream;

Некоторые члены абстрактного класса TextWriter:
```csharp
Член                Описание
Peek()              Дает следующий доступный символ в виде целого числа, не изменяя текущей позиции средства чтения. Значение -1 указывает на конец файла.
Read()              Читает данные из входного потока
ReadBlock()         Читает указанное максимальное количество символов из потока и записывает данные в буфер, с текущего индекса
ReadLine()          Читает строку символов из текущего потока и возвращает данные в виде строки (null - конец файла)
ReadToEnd()         Читает все символы от текущей позиции до конца потока и вертает их в виде строки
```
Пример чтения текстового файла:
```csharp
static void Main() 
{
    Console.WriteLine("Прочитанные строки:");
    //using (StreamReader reader = File.OpenText("text.txt"))
    using (StreamReader reader = new StreamReader("text.txt"))
    {
        string input = null;
        while ((input = reader.ReadLine()) != null)
        {
            Console.WriteLine(input);
        }
    }
    Console.WriteLine("Нажмите любую кнопку для завершения");
    Console.ReadKey();
}
```
Классы StringWriter и StringReader можно использовать для трактовки текста как потока символов в памяти, в основе находится объект класса StringBuilder.

Примеры использования:
```csharp
static void Main() 
{
    using (StringWriter stringWriter = new StringWriter())
    {
        stringWriter.Write("Пример текстовой строки");
        Console.WriteLine($"Содержимое потока: \"{stringWriter}\"");
        StringBuilder sb = stringWriter.GetStringBuilder();
        sb.Insert(0, "Дополнительно! ");
        Console.WriteLine($"StringBuilder: \"{sb.ToString()}\"");
        Console.WriteLine($"Содержимое потока после добавления: \"{stringWriter}\"");
        sb.Remove(0, "Дополнительно! ".Length);
        Console.WriteLine($"Содержимое потока после удаления: \"{stringWriter}\"");
        //читать данные из потока StringWriter
        using (StringReader stringReader = new StringReader(stringWriter.ToString()))
        {
            Console.WriteLine("Поток StringReader:");
            string input = null;
            while ((input = stringReader.ReadLine()) != null)
                Console.WriteLine(input);
        }
    }
    Console.WriteLine("Нажмите любую кнопку для завершения");
    Console.ReadKey();
}
```


## Подробное использование символьного потока 

Чтобы создать символьный поток, нужно поместить объект класса Stream (например, FileStream) «внутрь» объекта класса StreamWriter или объекта класса StreamReader. В этом случае байтовый поток будет автоматически преобразовываться в символьный. 
```csharp
StreamWriter(Stream stream);
```
Пример:
```csharp
StreamWriter fileOut=new StreamWriter(new FileStream("text.txt", FileMode.Create, FileAccess.Write));
```

Другой вид конструктора позволяет открыть поток сразу через обращение к файлу, где параметр name определяет имя открываемого файла:
```csharp
StreamWriter(string name, bool appendFlag);
```
Флаг - может принимать значение true, если нужно добавлять данные в конец файла, или false, если файл необходимо перезаписать.

В C# символы реализуются кодировкой Unicode. Чтобы было можно обрабатывать текстовые файлы, содержащие кириллические символы, созданные, например, в Блокноте, рекомендуется вызывать следующий вид конструктора StreamReader:
```csharp
StreamReader fileIn=new StreamReader ("c:\temp\data.txt", Encoding.GetEncoding(1251));
```
Параметр Encoding.GetEncoding(1251) говорит о том, что будет выполняться преобразование из кода Windows-1251 (одна из модификаций кода ASCII, содержащая кириллические символы) в Unicode.Encoding.GetEncoding(1251), реализованном в пространстве имён System.Text. 

Теперь для чтения данных из потока fileIn можно воспользоваться методом ReadLine. При этом, если будет достигнут конец файла, метод ReadLine вернёт значение null.

Пример простого использования:
```csharp
//запись
int[] arrInts = {1, 2, 3, 4, 5};
StreamWriter writer = new StreamWriter("data.txt");
for (int i = 0; i < arrInts.Length; i++)
{
    writer.WriteLine(arrInts[i]);
}
writer.Close();
//чтение
StreamReader reader = new StreamReader("data.txt");
int[] array = new int[5];
for (int i = 0; i < 5; i++)
{
    array[i] = int.Parse(reader.ReadLine());
}
reader.Close();
Array.ForEach(array, i => Write(i + " "));
```

Пример наипростого использования:
```csharp
string[] strs = {"1 первый", "2 второй"};
File.WriteAllLines("test.txt", strs);

string[] strs2 = File.ReadAllLines("test.txt");
Array.ForEach(strs2, WriteLine); //вывод элементов массива в консоль
```

## Двоичный поток

Двоичные файлы хранят данные в том же виде, в котором они представлены в оперативной памяти, то есть во внутреннем представлении. Двоичные файлы не применяются для просмотра человеком, они используются только для программной обработки.

Оба класса унаследованы прямо от System.Object.

Двоичный поток открывается на основе базового потока (например, FileStream). При этом двоичный поток будет преобразовывать байтовый поток в значения int-, double-, short-  и т.д. Классы BinaryWriter и BinaryReader, представляющие собой оболочки для FileStream и позволяющие преобразовывать байтовые потоки в двоичные для работы с int-, double-, short-  и т. д.

Некоторые члены класса BinaryWriter
```csharp
Член                Описание
BaseSteam           Свойство для чтения дает доступ к лежащему в основе потоку
Close()             Закрывает этот поток двоичный
Flush()             Выталкивает буфер двоичного потока
Seek()              Метод устанавливает позицию в текущем потоке
Write()             Метод записывает значение в текущий поток
```
Некоторые члены класса BinaryReader
```csharp
Член                Описание
BaseSteam           Свойство для чтения дает доступ к лежащему в основе потоку
Close()             Закрывает этот поток двоичный
PeekChar()          Метод возвращает следующий доступный символ без перемещения позиции потока
Read()              Метод читает заданный набор байтов или символов и сохраняет их во входном массиве
ReadXXX()           Методы чтения, которые извлекают из потока объекты различных типов
```
Пример записи и чтения нескольких типов данных в файл:
```csharp
static void Main()
{
    FileInfo fi = new FileInfo("test.dat");
    using (BinaryWriter bw = new BinaryWriter(fi.OpenWrite()))
    {
        Console.WriteLine($"Базовый поток: {bw.BaseStream}");
        double a1 = 1234.55;
        int a2 = 12655;
        string s1 = "A, B, C";
        bw.Write(a1);
        bw.Write(a2);
        bw.Write(s1);
    }
    Console.WriteLine("Запись завершена!");
    using (BinaryReader br = new BinaryReader(fi.OpenRead()))
    {
        Console.WriteLine(br.ReadDouble());
        Console.WriteLine(br.ReadInt32());
        Console.WriteLine(br.ReadString());
    }
    Console.WriteLine("Нажмите любую кнопку");
    Console.ReadKey();
}
```
Еще пример:
```csharp
//запись
using (FileStream f=new FileStream("data.bin",FileMode.Create))
{
    using (BinaryWriter fOut=new BinaryWriter(f))
    {
        for (int i=0; i<5; i++)
        {
            fOut.Write(i);
        }
    }
}
//чтение
using (FileStream f=new FileStream("data.bin",FileMode.Open))
{
    using (BinaryReader fIn=new BinaryReader(f))
    {
        for (int i=0; i<5; i++)
        {
            int a=fIn.ReadInt32();
            Console.Write(a+" ");
        }
    }
}
```
Двоичные потоки являются потоками с произвольным доступом, нумерация начинается с ноля. Произвольный метод обеспечивает метод Seek. 
```csharp
Seek(long newPos, SeekOrigin pos)
```

## Буферизированный поток

Самым быстрым способом чтения и записи данных является буферизированный поток.
```csharp
//запись
using (FileStream fs = new FileStream("data.bin", FileMode.Create, FileAccess.Write))
{
    int countPart = 4;
    int bufSize = (int) (8 / countPart);
    byte[] buffer = new byte[8];
    for (int i = 0; i < buffer.Length; i++)
    {
        buffer[i] = (byte)i;
    }
    using (BufferedStream bs = new BufferedStream(fs, bufSize))
    {
        for (int i = 0; i < countPart; i++)
        {
            bs.Write(buffer, i * bufSize, bufSize);
        }
    }
}
//чтение
using (FileStream fs = new FileStream("data.bin", FileMode.Open, FileAccess.Read))
{
    int countPart = 4;
    int bufSize = (int)fs.Length / countPart;
    byte[] arr = new byte[fs.Length];
    BufferedStream reader = new BufferedStream(fs);
    for (int i = 0; i < countPart; i++)
    {
        reader.Read(arr, i * bufSize, bufSize);
    }
    foreach (byte el in arr)
    {
        Write(el + " ");
    }
}
```

