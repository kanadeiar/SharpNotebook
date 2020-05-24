# Основы WPF Сервисы

## Web-сервис

"Веб-приложение ASP.NET", Пустой.

Веб-служба (ASMX).

Пример простого веб сервиса:
```csharp
[WebService(Namespace = "http://tempuri.org/")]
[WebServiceBinding(ConformsTo = WsiProfiles.BasicProfile1_1)]
[System.ComponentModel.ToolboxItem(false)]
public class WebService1 : System.Web.Services.WebService
{

    [WebMethod]
    public string HelloWorld()
    {
        return "Привет всем!";
    }
    [WebMethod]
    public string Summ(string number1, string number2)
    {
        float a,b;
        bool f1 = float.TryParse(number1,out a);
        if (f1 == false) return "В первом поле должно быть число";
        bool f2 = float.TryParse(number2,out b);
        if (f2 == false) return "Во втором поле должно быть число";
        return "Сумма: " + (a+b).ToString();
    }
}
```

Пример простого клиента этого веб сервиса (ASMX Client):
```csharp
<Window x:Class="WpfApp1WCFClient.MainWindow"
...
        Title="MainWindow" Height="400" Width="600" Background="AliceBlue">
    <Grid>
        <Grid.ColumnDefinitions><ColumnDefinition Width="Auto"/><ColumnDefinition Width="Auto"/></Grid.ColumnDefinitions>
        <Grid.RowDefinitions><RowDefinition Height="Auto"/><RowDefinition Height="Auto"/></Grid.RowDefinitions>
        <TextBox Text="1" x:Name="textBox1" Grid.Column="0" HorizontalAlignment="Center" Height="23" TextWrapping="Wrap" VerticalAlignment="Center" Width="160" Margin="10"/>
        <TextBox Text="2" x:Name="textBox2" Grid.Column="1" HorizontalAlignment="Center" Height="23" TextWrapping="Wrap" VerticalAlignment="Center" Width="160" Margin="10"/>
        <Button Click="summButton_Click" x:Name="summButton" Grid.Row="1" Content="Вычислить" HorizontalAlignment="Center" VerticalAlignment="Center" Width="100" Margin="10"/>
        <TextBlock Text="Text" x:Name="resultTextBlock" Grid.Column="1" Grid.Row="1" HorizontalAlignment="Center" VerticalAlignment="Center" TextWrapping="Wrap" Margin="10"/>
    </Grid>
</Window>
public partial class MainWindow : Window
{
    public MainWindow()
    {
        InitializeComponent();
    }

    private void summButton_Click(object sender, RoutedEventArgs e)
    {
        ServiceReference1.WebService1SoapClient serviceClient = new ServiceReference1.WebService1SoapClient();
        string sum = serviceClient.Summ(textBox1.Text,textBox2.Text);
        resultTextBlock.Text = sum;
    }
}
```

## WCF-сервис

"Приложение службы WCF".

Пример простого WCF-сервиса:
```csharp
//файл IService1.cs
public interface IService1
{
...
    [OperationContract]
    int CalculateDays(int day, int mounth, int year);
}
//файл Service1.svc.cs
public class Service1 : IService1
{
...
    public int CalculateDays(int day, int mounth, int year)
    {
        DateTime dt = new DateTime(year,mounth,day);
        int datetodays = DateTime.Now.Subtract(dt).Days;
        return datetodays;
    }
}
```
Для отладки служб - выделить "Service1.svc" и нажать F5. Путь к службе: «Файл» – «Добавить службу».

Пример простого клиента WCF-сервиса:
```csharp
<Window x:Class="WpfApp1WCFClient.MainWindow"
...
        Title="MainWindow" Height="400" Width="600" Background="AliceBlue">
    <Grid>
        <Grid.ColumnDefinitions><ColumnDefinition Width="Auto"/><ColumnDefinition Width="Auto"/></Grid.ColumnDefinitions>
        <Grid.RowDefinitions><RowDefinition/><RowDefinition/><RowDefinition/><RowDefinition/></Grid.RowDefinitions>
        <TextBlock Text="День" Grid.Column="0" Grid.Row="0" HorizontalAlignment="Center" TextWrapping="Wrap" VerticalAlignment="Center" Margin="10"/>
        <TextBlock Text="Месяц" Grid.Column="0" Grid.Row="1" HorizontalAlignment="Center" TextWrapping="Wrap" VerticalAlignment="Center" Margin="10"/>
        <TextBlock Text="Год" Grid.Column="0" Grid.Row="2" HorizontalAlignment="Center" TextWrapping="Wrap" VerticalAlignment="Center" Margin="10"/>
        <TextBox x:Name="dayTextBox" Grid.Column="1" Grid.Row="0" HorizontalAlignment="Center" Height="26" VerticalAlignment="Center" Width="200" Margin="10"/>
        <TextBox x:Name="monthTextBox" Grid.Column="1" Grid.Row="1" HorizontalAlignment="Center" Height="26" VerticalAlignment="Center" Width="200" Margin="10"/>
        <TextBox x:Name="yearTextBox" Grid.Column="1" Grid.Row="2" HorizontalAlignment="Center" Height="26" VerticalAlignment="Center" Width="200" Margin="10"/>
        <Button Click="Button_Click" Grid.Row="3" Content="Вычислить" HorizontalAlignment="Center" Width="100" Margin="10"/>
        <TextBlock x:Name="resultTextBlock" Text="Дней: " Grid.Column="1" Grid.Row="3" HorizontalAlignment="Center" TextWrapping="Wrap" VerticalAlignment="Center" Margin="10"/>
    </Grid>
</Window>
public partial class MainWindow : Window
{
...
    private void Button_Click(object sender, RoutedEventArgs e)
    {
        ServiceReference1.Service1Client service = new ServiceReference1.Service1Client();
        try
        {
            int day = Convert.ToInt32(dayTextBox.Text);
            int month = Convert.ToInt32(monthTextBox.Text);
            int year = Convert.ToInt32(yearTextBox.Text);
            resultTextBlock.Text = "Дней: " + service.CalculateDays(day,month,year);
        }
        catch (Exception ex)
        {
            MessageBox.Show($"Ошибка!\n"+ex.Message);
        }
    }
}
```

## Web-API сервис

"Веб-приложение ASP.NET",Пустой,Галка "Web API"

Запуск нужен одновременный двух проектов.

Пример простого Web API-сервиса:
```csharp
//Product.cs в папке Models
public class Product
{
    public int Id {get;set;}
    public string Name {get;set;}
    public string Category {get;set;}
    public decimal Price {get;set;}
}
//Добавленный API контроллер в папку Controllers ProductController.cs
public class ProductsController : ApiController
{
    Product[] products = new Product[]
    {
        new Product {Id=1,Name="Цикорий",Category="Бакалея",Price=100},
        new Product {Id=2,Name="Чебурашка",Category="Игрушки",Price=150},
        new Product {Id=3,Name="Молоток",Category="Инструменты",Price=50},
    };
    public IEnumerable<Product> GetAllProducts() => products;
    public IHttpActionResult GetProduct(int id)
    {
        var product = products.FirstOrDefault((p) => p.Id == id);
        if (product == null)
        {
            return NotFound();
        }
        return Ok(product);
    }
}
```
Проверка: https://localhost:44326/api/products в ответе должен быть XML файл.

Пример простого клиента Web API-сервиса (добавить NuGet пакет Microsoft.AspNet.WebApi.Client):

```csharp
//Product.cs
public class Product
{
    public int Id {get;set;}
    public string Name {get;set;}
    public string Category {get;set;}
    public decimal Price {get;set;}
}
//окно MainWindow
<Window x:Class="WpfApp1WCFClient.MainWindow"
...
        Loaded="Window_Loaded" Title="MainWindow" Height="400" Width="600" Background="AliceBlue">
    <Grid>
        <Grid.ColumnDefinitions><ColumnDefinition Width="*"/><ColumnDefinition Width="*"/></Grid.ColumnDefinitions>
        <Grid.RowDefinitions><RowDefinition Height="5*"/><RowDefinition Height="Auto"/></Grid.RowDefinitions>
        <DataGrid ItemsSource="{Binding}" x:Name="productDataGrid" Grid.ColumnSpan="2" AutoGenerateColumns="False" EnableRowVirtualization="True" Margin="10" IsReadOnly="True" FontSize="14">
            <DataGrid.Columns>
                <DataGridTextColumn Binding="{Binding Id}" x:Name="idColumn" Header="Id" IsReadOnly="True" Width="Auto"/>
                <DataGridTextColumn Binding="{Binding Name}" x:Name="nameColumn" Header="Название" IsReadOnly="True" Width="Auto"/>
                <DataGridTextColumn Binding="{Binding Category}" x:Name="categoryColumn" Header="Категория" IsReadOnly="True" Width="Auto"/>
                <DataGridTextColumn Binding="{Binding Price}" x:Name="priceColumn" Header="Цена" IsReadOnly="True" Width="Auto"/>
            </DataGrid.Columns>
        </DataGrid>
        <TextBlock Text="Id товара" Grid.Row="1" Margin="10" HorizontalAlignment="Center" VerticalAlignment="Center"/>
    </Grid>
</Window>
public partial class MainWindow : Window
{
    static HttpClient client;
    public MainWindow()
    {
        InitializeComponent();
        client = new HttpClient();
        client.BaseAddress = new Uri(@"https://localhost:44326/"); //uri адрес службы
    }
    private async void Window_Loaded(object sender, RoutedEventArgs e)
    {
        var products = await GetProductsAsync(client.BaseAddress + "api/products");
        productDataGrid.ItemsSource = products;
    }
    static async Task<IEnumerable<Product>> GetProductsAsync(string path)
    {
        IEnumerable<Product> products = null;
        try
        {
            var response = await client.GetAsync(path);
            response.EnsureSuccessStatusCode();
            products = await response.Content.ReadAsAsync<IEnumerable<Product>>();
        }
        catch (Exception ex)
        {
            MessageBox.Show("Ошибка!"+ex.Message);
        }
        return products;
    }
}
```





