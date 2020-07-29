# ADO.NET Автономный уровень Примеры

В WindowsForms есть элемент управления DataGridView, обладающий возможностью вывода содержимого DataSet и DataTable несколькими строками кода.

## Работа с DataSet

Пример простого приложения WF манипулирующего различными объектами DataSet:
```csharp
namespace WindowsFormsApp1
{
    public partial class FormMain : Form
    {
        List<Person> list;
        DataTable table;
        DataView view;
        public FormMain()
        {
            InitializeComponent();
            list = new List<Person>
            {
                new Person {Id = 1, Fam = "Testin1", Name = "Test1", Age = 18},
                new Person {Id = 2, Fam = "Testin2", Name = "Test2", Age = 28},
                new Person {Id = 3, Fam = "Testin3", Name = "Test3", Age = 20},
                new Person {Id = 4, Fam = "Testin2", Name = "Test4", Age = 12},
                new Person {Id = 5, Fam = "Testin2", Name = "Test1", Age = 23},
                new Person {Id = 6, Fam = "Testin2", Name = "Test2", Age = 24},
                new Person {Id = 7, Fam = "Testin1", Name = "Test3", Age = 18},
                new Person {Id = 8, Fam = "Testin2", Name = "Test4", Age = 28},
            };
            table = CreateDataTable(list);
            dataGridViewPersons.DataSource = table;
            view = CreateDataView20(table);
            dataGridView20.DataSource = view;
        }
        private DataTable CreateDataTable(ICollection<Person> list)
        {
            var table = new DataTable();
            var IdColumn = new DataColumn("Id", typeof(int));
            var FamColumn = new DataColumn("Fam", typeof(string));
            var NameColumn = new DataColumn("Name", typeof(string));
            var AgeColumn = new DataColumn("Age", typeof(int));
            table.Columns.AddRange(new[] { IdColumn, FamColumn, NameColumn, AgeColumn });
            foreach (var el in list)
            {
                var newRow = table.NewRow();
                newRow["Id"] = el.Id;
                newRow["Fam"] = el.Fam;
                newRow["Name"] = el.Name;
                newRow["Age"] = el.Age;
                table.Rows.Add(newRow);
            }
            return table;
        }
        private DataView CreateDataView20(DataTable table)
        {
            DataView retMe = new DataView(table);
            retMe.RowFilter = "Age > 20";
            return retMe;
        }
        //удаление строки
        private void buttonDelete_Click(object sender, EventArgs e)
        {
            try
            {
                DataRow rowToDelete = table.Select($"Id = {int.Parse(textBoxDeleteRowNumber.Text)}").First();
                rowToDelete.Delete();
                table.AcceptChanges();
            }
            catch (Exception ex)
            {
                MessageBox.Show(ex.Message);
            }
        }
        //показ отфильтрованных и отсортированных данных
        private void buttonFilter_Click(object sender, EventArgs e)
        {
            //фильтровать по фамилии и номерам ид больше трех
            string filterStr = $"Fam = '{textBoxFilterFam.Text}' AND Id > 3";
            //сортировать по имени в обратном порядке
            DataRow[] fams = table.Select(filterStr, "Name DESC");
            if (fams.Length == 0)
                MessageBox.Show("Строк с такими фамилиями не найдено!", "Фиговый результат");
            else
            {
                StringBuilder sb = new StringBuilder();
                foreach (var el in fams)
                    sb.AppendLine(el["Name"].ToString());
                MessageBox.Show($"Найдены {fams.Length} штук:\n" + sb.ToString(), "Результаты");
            }
        }
        //замена имени
        private void buttonReplaceName_Click(object sender, EventArgs e)
        {
            try
            {
                string filterStr = $"Name = '{textBoxReplaceNameFrom.Text}'";
                DataRow[] names = table.Select(filterStr);
                string strTo = textBoxReplaceNameTo.Text;
                foreach (var el in names)
                    el["Name"] = strTo;
            }
            catch (Exception ex)
            {
                MessageBox.Show(ex.Message);
            }
        }
    }
    //класс человек
    public class Person
    {
        public int Id { get; set; }
        public string Fam { get; set; }
        public string Name { get; set; }
        public int Age { get; set; }
    }
}
```

Пример работы с DataSet с несколькими таблицами взаимосвязанными:

Код SQL генерации тестовой базы данных:
```csharp
CREATE TABLE [dbo].[Person] (
    [Id]   INT           IDENTITY (1, 1) NOT NULL,
    [Fam]  NVARCHAR (64) NOT NULL,
    [Name] NVARCHAR (64) NOT NULL,
    [Age]  INT           NOT NULL,
    CONSTRAINT [PK_Id_Person] PRIMARY KEY CLUSTERED ([Id] ASC)
);
CREATE TABLE [dbo].[Child] (
    [Id]       INT          IDENTITY (1, 1) NOT NULL,
    [Fam]      NVARCHAR (64) NOT NULL,
    [Name]     NVARCHAR (64) NOT NULL,
	[Age] INT NOT NULL,
    [PersonId] INT          NOT NULL,
    CONSTRAINT [PK_Id_Child] PRIMARY KEY CLUSTERED ([Id] ASC),
    CONSTRAINT [FK_Child_To_Person] FOREIGN KEY ([PersonId]) REFERENCES [dbo].[Person] ([Id])
);

INSERT INTO Person([Fam], [Name], [Age])
VALUES (N'Иванов', N'Иван', 20),
(N'Петров', N'Петр', 22),
(N'Сидоров', N'Сидор', 18);
INSERT INTO Child([Fam], [Name], [Age], [PersonId])
VALUES (N'Михаилов', N'Михаил', 3, 1),
(N'Навозов', N'Дмитрий', 13, 1),
(N'Логинов', N'Логин', 3, 2);
```

Код конфигурационного файла и приложения:
```csharp
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
...
    <connectionStrings>
      <add name="SampleProvider" connectionString="Data Source=(localdb)\MSSQLLocalDB;Initial Catalog=sample;Integrated Security=True"/>
    </connectionStrings>
</configuration>
namespace WindowsFormsApp1
{
    public partial class FormMain : Form
    {
        private DataSet _dataSet = new DataSet("sample");
        private SqlDataAdapter _personAdapter;
        private SqlDataAdapter _childAdapter;
        private string _connectionString;
        public FormMain()
        {
            InitializeComponent();

            _connectionString = ConfigurationManager.ConnectionStrings["SampleProvider"].ConnectionString;
            _personAdapter = new SqlDataAdapter("SELECT * FROM Person", _connectionString);
            _childAdapter = new SqlDataAdapter("SELECT * FROM Child", _connectionString);
            var cbPerson = new SqlCommandBuilder(_personAdapter);
            var cbChild = new SqlCommandBuilder(_childAdapter);

            _personAdapter.Fill(_dataSet, "Person");
            _childAdapter.Fill(_dataSet, "Child");

            DataRelation relation = new DataRelation("PersonChild",
                _dataSet.Tables["Person"].Columns["Id"], 
                _dataSet.Tables["Child"].Columns["PersonId"]);
            _dataSet.Relations.Add(relation);

            dataGridViewPerson.DataSource = _dataSet.Tables["Person"];
            dataGridViewChild.DataSource = _dataSet.Tables["Child"];
        }
        private void buttonUpdateDatabase_Click(object sender, EventArgs e)
        {
            try
            {
                _personAdapter.Update(_dataSet, "Person");
                _childAdapter.Update(_dataSet, "Child");
            }
            catch (Exception ex)
            {
                MessageBox.Show(ex.Message);
            }
        }
        /// <summary> Инфа по родителю </summary>
        private void buttonGetPersonInfo_Click(object sender, EventArgs e)
        {
            StringBuilder sb = new StringBuilder();
            int id = int.Parse(textBoxPersonId.Text);
            var person = _dataSet.Tables["Person"].Select($"Id = {id}").First();
            sb.AppendLine($"Person {person["Id"]} : {person["Fam"].ToString().Trim()} {person["Name"].ToString().Trim()}");

            var childs = person.GetChildRows(_dataSet.Relations["PersonChild"]);
            sb.AppendLine("Дети:");
            if (childs.Length == 0)
                sb.AppendLine("отсутствуют");
            else
                sb.AppendLine($"{childs.Length} шт.");
            foreach (var c in childs)
            {
                sb.AppendLine($"Child {c["Id"]} : {c["Fam"].ToString().Trim()} {c["Name"].ToString().Trim()}");
            }
            MessageBox.Show(sb.ToString(), "Информация по родителю");
        }
        /// <summary> Инфа по ребенку </summary>
        private void buttonChildInfo_Click(object sender, EventArgs e)
        {
            StringBuilder sb = new StringBuilder();
            int id = int.Parse(textBoxChildId.Text);
            var child = _dataSet.Tables["Child"].Select($"Id = {id}").First();
            sb.AppendLine($"Child {child["Id"]} : {child["Fam"].ToString().Trim()} {child["Name"].ToString().Trim()}");

            var person = child.GetParentRow(_dataSet.Relations["PersonChild"]);
            sb.AppendLine("Родитель:");
            sb.AppendLine($"Person {person["Id"]} : {person["Fam"].ToString().Trim()} {person["Name"].ToString().Trim()}");

            MessageBox.Show(sb.ToString(), "Информация по ребенку");
        }
    }
}
```


## Пример работы с адаптером данных

Код SQL генерации тестовой базы данных:
```csharp
CREATE TABLE [dbo].[Person] (
    [Id]   INT           IDENTITY (1, 1) NOT NULL,
    [Fam]  NVARCHAR (64) NOT NULL,
    [Name] NVARCHAR (64) NOT NULL,
    [Age]  INT           NOT NULL,
    CONSTRAINT [PK_Id_Person] PRIMARY KEY CLUSTERED ([Id] ASC)
);
CREATE TABLE [dbo].[Child] (
    [Id]       INT          IDENTITY (1, 1) NOT NULL,
    [Fam]      NVARCHAR (64) NOT NULL,
    [Name]     NVARCHAR (64) NOT NULL,
	[Age] INT NOT NULL,
    [PersonId] INT          NOT NULL,
    CONSTRAINT [PK_Id_Child] PRIMARY KEY CLUSTERED ([Id] ASC),
    CONSTRAINT [FK_Child_To_Person] FOREIGN KEY ([PersonId]) REFERENCES [dbo].[Person] ([Id])
);

INSERT INTO Person([Fam], [Name], [Age])
VALUES (N'Иванов', N'Иван', 20),
(N'Петров', N'Петр', 22),
(N'Сидоров', N'Сидор', 18);
INSERT INTO Child([Fam], [Name], [Age], [PersonId])
VALUES (N'Михаилов', N'Михаил', 3, 1),
(N'Навозов', N'Дмитрий', 13, 1),
(N'Логинов', N'Логин', 3, 2);
```

Пример консольного приложения с адаптером базы данных:
```csharp
static void Main(string[] args)
{
    string connectionString = @"Data Source=(localdb)\MSSQLLocalDB;Initial Catalog=sample;Integrated Security=True";
    DataSet data = new DataSet("sample");
    SqlDataAdapter adapter = new SqlDataAdapter("SELECT * FROM Person", connectionString);
    adapter.Fill(data, "Person");
    PrintDataSet(data);
    Console.WriteLine("Нажмите любую кнопку ...");
    Console.ReadKey();
}
private static void PrintDataSet(DataSet dataSet)
{
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
}
```

Пример усовершенствованого выше консольного приложения с адаптером базы данных:
```csharp
static void Main(string[] args)
{
    string connectionString = @"Data Source=(localdb)\MSSQLLocalDB;Initial Catalog=sample;Integrated Security=True";
    DataSet data = new DataSet("sample");
    //SqlDataAdapter adapter = new SqlDataAdapter("SELECT * FROM Person", connectionString);
    var adapter = ConfigureNewAdapter(connectionString);
    //дружественные к пользователю имена колонок
    DataTableMapping tableMapping = adapter.TableMappings.Add("Fam","Family");
    tableMapping.ColumnMappings.Add("Name", "Person Name");
    tableMapping.ColumnMappings.Add("Age", "Person Ages");
    //int count = adapter.Fill(data, "Person");
    data.Tables.Add(GetAllPerson(adapter, out int count));
    Console.WriteLine($"Кол-во строк: {count}");
    PrintDataSet(data);
    DataRow row = data.Tables["Person"].Select("Id = 1").First();
    row["Name"] = "test1"; //изменение
    UpdatePerson(adapter, data.Tables["Person"]);
    Console.WriteLine("Изменение данных:");
    PrintDataSet(data);
    Console.WriteLine("Нажмите любую кнопку ...");
    Console.ReadKey();
}
/// <summary> Создание нового адаптера со всеми командами </summary>
private static SqlDataAdapter ConfigureNewAdapter(string connectionString)
{
    var adapter = new SqlDataAdapter("SELECT * FROM Person", connectionString);
    var _ = new SqlCommandBuilder(adapter); //конфигурирование остальных команд
    return adapter;
}
/// <summary> Заполнение только одной таблицы адаптером данных </summary>
private static DataTable GetAllPerson(SqlDataAdapter adapter, out int count)
{
    DataTable table = new DataTable("Person");
    count = adapter.Fill(table); //заполнение таблицы данными
    return table;
}
/// <summary> Обновление данных в таблице </summary>
private static void UpdatePerson(SqlDataAdapter adapter, DataTable table)
{
    adapter.Update(table); //обновление хранилища по таблице
}
private static void PrintDataSet(DataSet dataSet)
{
//то-же самое
}
```

## Пример приложения с использованием визуального конструирования набора базы данных

Пример с использованием строго типизированных автоматически сгенерированных наборов данных (автоматически сгеннерированные наборы данных находятся отдельно в библиотеке ClassLibrary1DAL):
```csharp
using ClassLibrary1DAL;
using ClassLibrary1DAL.sampleDataSetTableAdapters;

namespace ConsoleApp1
{
    class Program
    {
        static void Main(string[] args)
        {
            var table = new sampleDataSet.PersonDataTable();
            var adapter = new PersonTableAdapter();
            adapter.Fill(table);
            PrintPerson(table);
            Console.WriteLine("Изменение данных в хранилище данных");
            AddPersons(table, adapter);
            table.Clear();
            adapter.Fill(table);
            PrintPerson(table);
            Console.WriteLine("Удаление данных в хранилище данных\nИд удаляемой строки:");
            if (int.TryParse(Console.ReadLine(), out int idDel))
            {
                DeletePerson(table, adapter, idDel);
            }
            table.Clear();
            adapter.Fill(table);
            PrintPerson(table);
            CallStoredProc();
            Console.WriteLine("Нажмите любую кнопку ...");
            Console.ReadKey();
        }
        public static void AddPersons(sampleDataSet.PersonDataTable table, PersonTableAdapter adapter)
        {
            try
            {
                var newRow = table.NewPersonRow();
                newRow.Fam = "TestingX";
                newRow.Name = "TestX";
                newRow.Age = 20;
                table.AddPersonRow(newRow);
                table.AddPersonRow("TestingX2", "TestX2", 21);
                adapter.Update(table); //обновить базу данных
            }
            catch (Exception ex)
            {
                Console.WriteLine(ex.Message);
            }
        }
        public static void DeletePerson(sampleDataSet.PersonDataTable table, PersonTableAdapter adapter, int id)
        {
            try
            {
                var delRow = table.FindById(id);
                adapter.Delete(delRow.Id, delRow.Fam, delRow.Name, delRow.Age);
            }
            catch (Exception ex)
            {
                Console.WriteLine(ex.Message);
            }
        }
        public static void CallStoredProc()
        {
            Console.WriteLine("Показ информации о персоне:");
            try
            {
                var queriesTableAdarter = new QueriesTableAdapter();
                Console.Write("Введите ид персоны: >");
                int id = int.Parse(Console.ReadLine());
                string name = string.Empty;
                queriesTableAdarter.GetName(id, ref name);
                Console.WriteLine($"Персона с ид {id} имеет имя {name}");
            }
            catch (Exception ex)
            {
                Console.WriteLine(ex.Message);
            }
        }
        public static void PrintPerson(sampleDataSet.PersonDataTable table)
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
    }
}
```





