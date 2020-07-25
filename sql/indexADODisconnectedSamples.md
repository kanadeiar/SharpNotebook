# ADO.NET Автономный уровень Примеры

В WindowsForms есть элемент управления DataGridView, обладающий возможностью вывода содержимого DataSet и DataTable несколькими строками кода.

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
