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

## Манипулирование различными простыми объектами DataSet

Добавление данных в DataTable:
```csharp
foreach (var el in list)
{
    var newRow = table.NewRow();
    newRow["Name"] = el.Name;
    table.Rows.Add(newRow);
}
```

Фильтр данных из DataTable:
```csharp
//фильтровать по фамилии и номерам ид больше трех
string filterStr = $"Fam = '{textBoxFilterFam.Text}' AND Id > 3";
//сортировать по имени в обратном порядке
DataRow[] fams = table.Select(filterStr, "Name DESC");
```

Изменение данных в DataTable путем фильра строк и их изменение:
```csharp
string filterStr = $"Name = '{textBoxReplaceNameFrom.Text}'";
DataRow[] names = table.Select(filterStr);
string strTo = textBoxReplaceNameTo.Text;
foreach (var el in names)
    el["Name"] = strTo;
```

Удаление строк из DataTable путем поиска строки через фильр и установку свойства:
```csharp
DataRow rowToDelete = table.Select($"Id = {int.Parse(textBoxDeleteRowNumber.Text)}").First();
rowToDelete.Delete();
table.AcceptChanges();
```

## Объекты DataSet с несколькими таблицами и отношениями между ними

Истинная мощь автономного уровня появляется, когда в DataSet множество взаимосвязанных таблиц DataTable, при этом в коллекции DataRelations содержатся взаимосвязи между таблицами DataRelation. Объекты DataRelation можно применять чтоб на клиентском уровне перемещаться между объктами по их линиям связи, автономно.

Связи между таблицами:
```csharp
DataRelation relation = new DataRelation("PersonChild",
    _dataSet.Tables["Person"].Columns["Id"], 
    _dataSet.Tables["Child"].Columns["PersonId"]);
_dataSet.Relations.Add(relation);
```

Перемещение по связям:
```csharp
var childs = person.GetChildRows(_dataSet.Relations["PersonChild"]);
var person = child.GetParentRow(_dataSet.Relations["PersonChild"]);
```

## Адаптер данных

Адаптер данных - класс, применяемый для заполнения DataSet объектами DataTable и отбравки обратно в хранилище измененных объектов.

Некторые члены адаптера DbDataAdapter:

Член                     | Описание
-------------------------|-------------------------
Fill()                   | Выполняет SQL команду SELECT выборки данных из базы данных в объект/объекты DataTable
SelectCommand InsertCommand UpdateCommand DeleteCommand | Установка SQL комманд, которые будут отправляться к базе данных при выполнении методов Fill() и Update()
Update()                 | Выполняет SQL команды INSERT, UPDATE, DELETE для сохранения в базе данных изменений из DataTable

У адаптеров подключение и отключение от базы данных происходит автоматически. Но адаптеру по прежнему нужно передать действительный объект подключения или объект подключения, чтобы сообщить, к какой базе данных нужно подключаться. Адаптер независим от баз данных и может налету подключать разные команды, объекты и разные типы баз данных (MSSQL, MySQL, Oracle).

Пример работы с адаптером базы данных:
```csharp
string connectionString = @"Data Source=(localdb)\MSSQLLocalDB;Initial Catalog=sample;Integrated Security=True";
DataSet data = new DataSet("sample");
SqlDataAdapter adapter = new SqlDataAdapter("SELECT * FROM Person", connectionString); //адаптер с соединением и сразу с командой
int count = adapter.Fill(data, "Person"); //заполнение только таблицы Person
```

Чтобы упростить работу с адаптерами данных, каждый поставщик данных ADO.NET снабжается построителем команд (SqlCommandBuilder), который автоматически генерирует InsertCommand UpdateCommand DeleteCommand адаптера на основе установленного SelectCommand.

Способен строить команды только если выполняются условия:

- SQL команда взаимодействует только с одной таблицей

- единственная таблица имеет первичный ключ

- таблица должна иметь столбец или столбцы первичного ключа включенным в SQL оператор SELECT.

Пример применения построителя команд с адаптером:
```csharp
var adapter = new SqlDataAdapter("SELECT * FROM Person", connectionString);
var _ = new SqlCommandBuilder(adapter); //конфигурирование остальных команд
```

## Визуальное конструирование базы данных

### Конструирование набора базы данных интрументами Windows Forms

Используя визуальные конструкторы, поддерживаемые элементом управления DataGridView из WindowsForms, можно визуально создавать код доступа к базе данных прямо в кодовую базу графического интерфейса.

С элемента DataGridView доступен мастер конфигурирования источников данных. 

По резельтатам работы мастера, после последовательности шагов, будут созданы в лотке с компонентами три компонента для доступа к базе данных: DataSet, BindingSource, TableAdapter. IDE автоматически создаст большой объем кода и настроит элемент управления для последующего использования.

Мастер помимо изменений в конфигурационный файл создает строго типизированный класс DataSet. Это класс, расширяющий стандартный DataSet и открывающий доступ к нескольким членам, которые позволяют работать с базой данных напрямую, не углубляясь в коллекцию таблиц с использованием свойства Tables.

В коде окна мастер создаст метод с кодом загрузки данных из хранилища в таблицу целевую при запуске приложения:
```csharp
private void FormMain_Load(object sender, EventArgs e)
{
    // TODO: данная строка кода позволяет загрузить данные в таблицу "sampleDataSet.Person". При необходимости она может быть перемещена или удалена.
    this.personTableAdapter.Fill(this.sampleDataSet.Person);
}
```

Можно дополнить кнопкой, которая будет сохранять изменения в таблице в хранилище данных и извлекать оттуда обратно данные:
```csharp
private void buttonRefresh_Click(object sender, EventArgs e)
{
    try
    {
        this.personTableAdapter.Update(this.sampleDataSet.Person);
    }
    catch (Exception ex)
    {
        MessageBox.Show(ex.Message);
    }
    this.personTableAdapter.Fill(this.sampleDataSet.Person);
}
```

### Конструирование набора базы данных визуальным конструктором набора данных

Инструменты визуального конструирования набора базы данных можно активировать в лобой разновидности проекта. 

При активации добавления в проект нового элемента "DataSet" запускается визуальный конструктор набора данных. Нужно перетащить нужные таблицы в область конструктора из обозревателя серверов, связи между таблицами будут учтены.

IDE автоматически создаст большой объем кода и настроит элемент управления для последующего использования.

Полученные таким образом строго типизированные классы можно применять в любом приложении .NET. 

Пример выборки данных из хранилища посредством строго типизированных классов сгенерированного набора данных:
```csharp
static void Main(string[] args)
{
    var table = new sampleDataSet.PersonDataTable();
    var adapter = new PersonTableAdapter();
    adapter.Fill(table);
    PrintPerson(table); //распечатка в консоль данных
    Console.WriteLine("Нажмите любую кнопку ...");
    Console.ReadKey();
}
```

Пример добавления тестовых данных в хранилище с помощью сгенерированного кода:
```csharp
var newRow = table.NewPersonRow();
newRow.Fam = "TestingX";
newRow.Name = "TestX";
newRow.Age = 20;
table.AddPersonRow(newRow);
table.AddPersonRow("TestingX2", "TestX2", 21); //другой способ
adapter.Update(table); //обновить базу данных
```

Пример удаления тестовых данных из хранилища с помощью сгенерированного кода:
```csharp
var delRow = table.FindById(1);
adapter.Delete(delRow.Id, delRow.Fam, delRow.Name, delRow.Age);
```

Пример вызова хранимой процедуры с помощью сгенерированного кода:
```csharp
int id = int.Parse(Console.ReadLine());
string name = string.Empty;
queriesTableAdarter.GetName(id, ref name);
Console.WriteLine($"Персона с ид {id} имеет имя {name}");
```

## Применение LINQ to DataSet

Этот метод наиболее удобный позволяет манипулировать данными DataSet используя выражения запросов LINQ. Следует знать что запросы LINQ применяются только к DataSet, а не напрямую к хранилищу данных. Расширяющий метод AsEnumerable объекта DataSet возвращает объект, содержащий коллекцию объектов DataRow, с которыми можно делать foreach & LINQ.

Расширяющий метод Field<>() типа DataRow позволяет привнести в запрос строгую типизацию, избавляя от возможных ошибок приведения типа к несовместимому типу.

Можно создавать новые объекты DataTable из запроса LINQ используя расширяющий метод CopyToDataTable(). А с помощью метода AsDataView() - объекты DataView.

Пример на основе строго типизированных классов сгенерированного набора данных:
```csharp
PersonTableAdapter adapter = new PersonTableAdapter();
sampleDataSet.PersonDataTable table = adapter.GetData();
//получение коллекции объектов
var enumTable = table.AsEnumerable(); 
Console.WriteLine("Список всех:");
foreach (var el in enumTable)
    Console.WriteLine($"Имя: {el.Name} возраст: {el.Age}");
//пример проецирования нового типа данных
var pers20 = table.AsEnumerable().Where(p => p.Age == 20).Select(p => new { p.Fam, p.Name });
Console.WriteLine("Все кому 20 лет:");
foreach (var el in pers20)
    Console.WriteLine($"{el.Fam} {el.Name}");
//пример безопасного проецирования нового типа данных
var pers20safety = table.AsEnumerable().Where(p => p.Field<int>("Age") == 21).Select(p => new { Fam = p.Field<string>("Fam"), Name = p.Field<string>("Name") });
Console.WriteLine("Все кому 21 год:");
foreach (var el in pers20)
    Console.WriteLine($"{el.Fam} {el.Name}");
//создание нового объекта DataTable
var personsNew = table.AsEnumerable().Where(p => p.Id > 5);
DataTable table2 = personsNew.CopyToDataTable();
Console.WriteLine("Новая таблица:");
var reader = table2.CreateDataReader();
while (reader.Read())
    Console.WriteLine($"Фамилия: {reader.GetValue(1).ToString().Trim()}");
```



