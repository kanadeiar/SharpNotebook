# Основы WPF Сервисы

## Web-сервис

Веб-приложение ASP.NET, Пустой.

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

Пример простого клиента этого веб сервиса:
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

Приложение службы WCF.

Пример простого WCF-сервиса:
```csharp

```

Пример простого клиента WCF-сервиса:
```csharp

```



