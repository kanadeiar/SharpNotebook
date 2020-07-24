# Автономный уровень ADO.NET

Автономный уровень позволяет моделировать данные из базы данных в памяти, внутри вызывающего кода, применением членов из пространства имен System.Data. Применением автономного уровня можно моделировать реляционные данные в памяти, включая табличные блоки строк и столбцов, отношения между таблицами, ограничения столбцов, первичные ключи, представления и прочее. К смоделированным данным можно применять фильтры, отправлять запросы в память и сохранять/загружать данные в формате XML и в двоичном формате.

Для выборки и обновления данных в базе данных нужно задействовать объект - адаптер данных (абстрактный класс DbDataAdapter). Объекты адаптера перемещают данные из вызывающего кода и источника данных с примененением объектов DataSet. Последний - это контейнер для любого количества объектов DataTable - этот объект содержит колекцию объектов DataColumn и DataRow. Адаптеры данных удерживают подключение открытым в течении минимально возможного времени. Физическая база данных не обновляется, пока не будет передат объект DataTable из DataSet адаптеру даных для обновления.

## Контейнер объектов базы данных DataSet

Этот объект содержит в себе коллекцию таблиц - свойство Tables. Можно программироваь отношения "родительский - дочерний" между таблицами, ограничение внешнего ключа через свойство Relations. А свойство ExtendedProperities позволяет создавать дополнительную информацию в виде пар "имя-значение".

Некоторые свойства DataSet:

Свойство                 | Описание
-------------------------|-------------------------
CaseSensitive            | Установка, что будет/нет чуствительно к регистру сравнение строк - по умолчанию false
DataSetName              | Дружественное имя объекта DataSet
EnforceConstrains        | Установка - соблюдаются ли правила ограничений при попытке выполнить любые операции обновления
HasErrors                | Получение значения что есть ошибки в любой строке любого объекта DataTable внутри DataSet
RemotingFormat           | Определение, каким образом DataSet должен сериализовать свое содержимое - в двоичном виде или XML

Некоторые методы DataSet:

Методы                   | Описание
-------------------------|-------------------------
AcceptChanges()          | Фиксирует все изменения, внесенные в текущий DataSet с момента его загрузки или последнего вызова AcceptChanges()
Clear()                  | Очищает данные DataSet, удаляя каждую строку из каждого DataTable
Clone()                  | Клонирует структуру, но не данные, включая каждую таблицу DataTable, отношение, ограничения
Copy()                   | Копирует структуру и данные
GetChanges()             | Копия, содержащая все изменения, внесенные в объект с момета загрузки или последнего фиксирования изменений. Можно получить тольно новые строки, только измененные строки или только удаленные строки.
HasChages()              | Значение, что объект был изменен, добавлены, изменены или удалены данные
Merge()                  | Объединяет текущий объект с указанным
ReadXml()                | Определение структуры объекта DataSet и заполнение его данными, основываясь на XML схеме данных из потока
RejectChanges()          | Откат всех изменений, внесенные в объект с момента загрузки или последнего фиксирования изменений.
WriteXml()               | Запись содержимого объекта DataSet в действитльный поток

Пример работы с DataSet (создание нового контейнера):
```csharp
DataSet dataSet = new DataSet("Sample");
dataSet.ExtendedProperties["TimeStamp"] = DateTime.Now; //время создания
dataSet.ExtendedProperties["DataSetId"] = Guid.NewGuid(); //уникальный идентификтор
dataSet.ExtendedProperties["Company"] = "Пример";
```

## Одиночный столбец DataColumn

Одиночный столбец внутри таблицы DataTable. На каждый столбец можно установить набор ограничений. Каждый столбец должен отображать лежащий в основе тип данных.

Столбец можно сделать автоинкрементированным. Значение такого столбца будет устанавливаться автоматически на основе предидущего значения.

Некоторые свойства DataColumn:

Свойство                | Описание
------------------------|-------------------------
AllowDBNull             | Указание, что столбец может содержать значения null
AutoIncrement, AutoIncrementSeed, AutoIncrementStep | Настройка поведения автоинкремента для заданного столбца. Полезно в случаях, когда нужно гарантировать уникальность значений.
Caption                 | Заголовок, отображаемый для столбца, дружественное имя для пользователя
ColumnMapping           | Представление объекта DataColumn при сохранении DataSet в документе XML. Возможное - записывание как элемента XML, атрибут XML, простое текстовое содержимое или игнорироваться
ColumnName              | Имя столбца в коллекции Columns объекта DataTable, стандартное значение - Coumn1, Column2, Column3 и т.д.
DataType                | Тип данных, хранящихся в столбце
DefaultValue            | Стандартное значение, присваиваемое при вставке новых строк
Expression              | Выражение для фильтрации строк, вычисления значения столбца или агрегированного столбца
Ordinal                 | Числовая позиция столбца в коллекции Colums объекта DataTable
ReadOnly                | Установка, что столбец доступен только для чтения, после добавления строки в таблицу, стандарт - false
Table                   | Вертает объект DataTable, содержащий текущий объект DataColumn
Unique                  | Значение, указывающее, должны ли значения этого столбца быть уникальными или разрешены одинаковые значения. При установке столбцу ограничения первичного ключа, это свойство должно быть true

## Строка данных DataRow

Представляет собой объект-строку данных таблицы DataTable. Создавать экземпляр DataRow напрямую невозможно, открытый конструктор отсутствует. Новый объект получается из объекта DataTable. После получения каждый столбец строки можно заполнять новыми данными через индексатор по имени или по номеру.

Некоторые члены DataRow:

Свойство                | Описание
------------------------|-------------------------
HasErrors, GetColumnInError(), GetColumnError(), ClearError(), RowError | HasError - вертает булевское значение, сигнал о ошибках в DataRow. С помощью второго метода можно получить проблемные столбцы, с помощью третьего - описание ошибки. Четвертый метод - очистка ошибок. RowError - настройка описания ошибки.
ItemArray               | Свойство значение всех столбцов в строке с применением массива объектов
RowState                | Определение текущего состояния объекта DataRow в содержащем его DataTable
Table                   | Свойство для получения ссылки на объект DataTable, содержащий данный объект DataRow
AcceptChanges(), RejectChanges() | Фиксация или отклонение изменений, внесенных в данную строку с момента последнего AcceptChanges()
BeginEdit(), EndEdit(), CancelEdit() | Начало, заканчивание и отмена редактирования строки DataRow
Delete()                | Пометка строки, которую необходимо удалить при фиксации изменений
IsNull()                | Вертает значение, указывающее, содержит ли этот столбец значение null

Свойство RowState служит для идентификации строк в таблице которые были изменены, добавлены, удалены. 

Свойству RowState можно установить значение из перечисления DataRowState:

Значение                | Описание
------------------------|-------------------------
Added                   | Строка была добавлена, а метод AcceptChanges() пока не вызывался
Deleted                 | Помечена для удаления методом Delete(), метод AcceptChanges() не вызывался
Detached                | Строка была создана, но не является частью какой-либо коллекции. Состояне строки после создания, до добавления в коллекцию. В таком же состоянии после удаления из коллекции.
Modified                | Строка измененена, но метод AcceptChanges() не вызывался
Unchanged               | Строка не менялась с момента вызова AcceptChanges()

Это позволяет объекту DataTable определять, какие строки были изменены и во премя передачи обновленой информации в хранилище данных, отправляются только измененные данные.

## Таблица данных DataTable

Некоторые члены DataTable:

Свойство                 | Описание
-------------------------|-------------------------
CaseSensitive            | Установка, что будет/нет чуствительно к регистру сравнение строк - по умолчанию false
ChildRelations           | Коллекция дочерних отношений этой таблицы DataTable
Constrains               | Коллекция ограничений таблицы
Copy()                   | Копирует структуру и данные из таблицы DataTable в новый такой-же объект
DataSet                  | Вертает объект DataSet, которая содержит эту таблицу
DefaultView              | Вертает настроенное представление таблицы, включающее отфильтрованое предтавление или позицию курсора
ParentRelations          | Вертает коллекцию родительских отношений для таблицы DataTable
PrimaryKey               | Массив столбцов, которые выступают в качестве первичных ключей для таблицы данных
TableName                | Имя таблицы. Можно устанавливать в конструкторе таблицы.

Пример работы с DataColumn, DataRow, DataTable, DataSet:
```csharp
var personIdColumn = new DataColumn("Id", typeof(int))
{
    Caption = "Id №",
    ReadOnly = true,
    AllowDBNull = false,
    Unique = true,
    AutoIncrement = true, //автоинкремент
    AutoIncrementSeed = 1,
    AutoIncrementStep = 1,
};
var personFamColumn = new DataColumn("Fam", typeof(string));
var personNameColumn = new DataColumn("Name", typeof(string));
var personAgeColumn = new DataColumn("Age", typeof(int));
var personTable = new DataTable("Person");
personTable.Columns.AddRange(new[] {personIdColumn, personFamColumn, personNameColumn, personAgeColumn});
DataRow person1 = personTable.NewRow();
person1["Fam"] = "Иванов";
person1["Name"] = "Иван";
person1["Age"] = 18;
personTable.Rows.Add(person1);
person1 = personTable.NewRow();
person1[1] = "Петров";
person1[2] = "Петр";
person1[3] = 28;
personTable.Rows.Add(person1);
//установка первичного ключа таблицы
personTable.PrimaryKey = new[] { personTable.Columns[0] };
dataSet.Tables.Add(personTable); //установка таблицы в контейнер
```

Пример вывода содержимого контейнера DataSet в консоль:
```csharp
Console.WriteLine($"DataSet id named: {dataSet.DataSetName}");
foreach (DictionaryEntry de in dataSet.ExtendedProperties)
    Console.WriteLine($"Ключ = {de.Key}, значение = {de.Value}");
Console.WriteLine("*************************************************");
foreach (DataTable table in dataSet.Tables)
{
    Console.WriteLine($"=> {table.TableName} Таблица:");
    for (var ci = 0; ci < table.Columns.Count; ci++)
        Console.Write($"{table.Columns[ci].ColumnName}\t");
    Console.WriteLine("\n------------------------------------------------");
    for (var ri = 0; ri < table.Rows.Count; ri++)
    {
        for (var ci = 0; ci < table.Columns.Count; ci++)
            Console.Write($"{table.Rows[ri][ci]}\t");
        Console.WriteLine();
    }
}
```

## Чтение данных из таблицы в памяти

Объект DataTable поддерживает метод CreateDataReader(), вертающий объект, позволяющий прочитывать данные в манере подключенного уровня доступа к данным. Используется единая модель независимо от того, какой уровень ADO.NET применяется для получения данных.

Пример чтения и вывода в консоль содержимого таблицы:
```csharp
private static void PrintTable(DataTable table)
{
    DataTableReader reader = table.CreateDataReader();
    while (reader.Read())
    {
        for (var ci = 0; ci < reader.FieldCount; ci++)
            Console.Write($"{reader.GetValue(ci).ToString().Trim()}\t");
        Console.WriteLine();
    }
    reader.Close();
}
```

## Сериализация

Сериализация возможна как в формат XML, так и в бинарный формат

Пример:
```csharp
private static void SaveAndLoadAsXml(DataSet sample)
{
    sample.WriteXml("sample.xml");
    sample.WriteXmlSchema("sample.xsd"); //файл схемы данных
    sample.Clear();
    sample.ReadXml("sample.xml");
}
private static DataSet SaveAndLoadAsBinary(DataSet sample)
{
    DataSet retMe;
    sample.RemotingFormat = SerializationFormat.Binary;
    var bFormat = new BinaryFormatter();
    using (var stream = new FileStream("binarysample.bin", FileMode.Create))
    {
        bFormat.Serialize(stream, sample);
    }
    using (var stream = new FileStream("binarysample.bin", FileMode.Open))
    {
        retMe = (DataSet)bFormat.Deserialize(stream);
    }
    return retMe;
}
```














