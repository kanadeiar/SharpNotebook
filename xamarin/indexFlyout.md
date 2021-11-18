# Летающая панель Xamarin.Forms

Flyout позволяет создавать одностраничные приложения.

## Базовая летающая панель

Необходимо изменить базовый класс главной страницы:

```xml
<Shell ...
             Title="Летающая панель"
             FlyoutWidth="300">
    <Shell.FlyoutHeader>
        <local:HeaderContentView/>
    </Shell.FlyoutHeader>
    <FlyoutItem Title="Тест приложения"
                Shell.TabBarIsVisible="False"
                FlyoutDisplayOptions="AsMultipleItems">
        <ShellContent Title="Test" Icon="icon_about.png" IsTabStop="True" ContentTemplate="{DataTemplate local:Page1}" />
        <ShellContent Title="Two" Icon="icon_feed.png" IsTabStop="True" ContentTemplate="{DataTemplate local:Page2}" />
    </FlyoutItem>
</Shell>
```

```csharp
public partial class MainPage : Shell
{
    public MainPage()
    {
        InitializeComponent();
    }
}
```

Заголовок:

```xml
<ContentView ...
             HeightRequest="180"
             x:Class="FlyoutApp.HeaderContentView">
    <Grid HorizontalOptions="FillAndExpand" VerticalOptions="FillAndExpand" BackgroundColor="#1978d0">
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="10"/>
            <ColumnDefinition Width="Auto"/>
            <ColumnDefinition Width="Auto"/>
            <ColumnDefinition Width="30"/>
        </Grid.ColumnDefinitions>
        <Grid.RowDefinitions>
            <RowDefinition Height="20" />
            <RowDefinition Height="80" />
            <RowDefinition Height="40" />
            <RowDefinition Height="Auto" />
        </Grid.RowDefinitions>

        <Image Source="favicon.png" Grid.Column="1" Grid.Row="1" HeightRequest="80" WidthRequest="80" HorizontalOptions="Start" VerticalOptions="End"/>

        <StackLayout VerticalOptions="CenterAndExpand" Grid.Column="2" Grid.Row="1" Grid.ColumnSpan="3" Orientation="Vertical">
            <Label Text="Приложение" TextColor="White" HorizontalTextAlignment="Start" VerticalTextAlignment="Center" />
            <Label Text="kanadeiar@gmail.com" TextColor="White" HorizontalTextAlignment="Start" VerticalTextAlignment="Center" />
        </StackLayout>

        <Label Text="Приложение с примером летающей панели" Grid.Column="1" Grid.Row="2" Grid.ColumnSpan="3" TextColor="White" HorizontalTextAlignment="Start" VerticalTextAlignment="Center" />        
    </Grid>
</ContentView>
```

Одна из страниц:

```xml
<ContentPage xmlns="http://xamarin.com/schemas/2014/forms"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="FlyoutApp.Page1">
    <ContentPage.Content>
        <StackLayout>
            <Label Text="I am page # 1"
                VerticalOptions="CenterAndExpand" 
                HorizontalOptions="CenterAndExpand" />
        </StackLayout>
    </ContentPage.Content>
</ContentPage>
```
