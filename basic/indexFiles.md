# Файлы

С#-программы выполняют операции ввода-вывода посредством потоков, которые построены на иерархии классов.

Центральную часть потоковой С#-системы занимает класс Stream пространства имен System.IO. Класс Stream представляет байтовый поток и является базовым для всех остальных потоковых классов.

## Классы байтовых потоков
Из класса Stream выведены такие байтовые классы потоков, как:

FileStream — байтовый поток, разработанный для файлового ввода-вывода. 

BufferedStream заключает байтовый поток в оболочку и добавляет буферизацию, которая во многих случаях увеличивает производительность программы. 

MemoryStream — байтовый поток, который использует память для хранения данных.

## Байтовый поток
Первонаперво создается объект класса FileStream.
```csharp
FileStream(string filename, FileMode mode, FileAccess how)
```
В нем указывается имя файла, режим открытия файла и способ доступа к файлу.

Для чтения очередного байта из потока, связанного с физическим файлом, используется метод ReadByte(). После прочтения очередного байта внутренний указатель перемещается на следующий байт файла. Если достигнут конец файла, метод ReadByte() возвращает значение -1.

Для побайтовой записи данных в поток используется метод WriteByte().

После работы файл необходимо закрыть.

Пример:
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

## Работа с текстовыми файлами - символьный поток
Поэтому для работы непосредственно с текстовыми файлами предназначены отдельные классы - StreamReader и StreamWriter из пространства имен System.IO. Классы StreamWriter и StreamReader, представляющие собой оболочки для FileStream и позволяющие преобразовывать байтовые потоки в символьные.

Использование: чтобы создать символьный поток, нужно поместить объект класса Stream (например, FileStream) «внутрь» объекта класса StreamWriter или объекта класса StreamReader. В этом случае байтовый поток будет автоматически преобразовываться в символьный. 
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

Двоичный поток открывается на основе базового потока (например, FileStream). При этом двоичный поток будет преобразовывать байтовый поток в значения int-, double-, short-  и т.д. Классы BinaryWriter и BinaryReader, представляющие собой оболочки для FileStream и позволяющие преобразовывать байтовые потоки в двоичные для работы с int-, double-, short-  и т. д.

Пример:
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

