# Навигация Xamarin.Forms

## Основы навигации

Пример навигации:

App.xaml.cs
```csharp
public App()
{
    InitializeComponent();
    MainPage = new NavigationPage(new MainPage());
}
public class CommonPage : ContentPage
{
    public CommonPage()
    {
        Title = "Common Page";
        Button backButton = new Button
        {
            Text = "Назад",
            HorizontalOptions = LayoutOptions.Center,
            VerticalOptions = LayoutOptions.Center,
        };
        backButton.Clicked += BackButton_Clicked;
        Content = backButton;
    }
    private async void BackButton_Clicked(object sender, EventArgs e)
    {
        await Navigation.PopAsync();
    }
}
public class ModalPage : ContentPage
{
    public ModalPage()
    {
        Title = "Modal Page";
        Button backButton = new Button
        {
            Text = "Назад",
            HorizontalOptions = LayoutOptions.Center,
            VerticalOptions = LayoutOptions.Center,
        };
        backButton.Clicked += BackButton_Clicked;
        Content = backButton;
    }
    private async void BackButton_Clicked(object sender, EventArgs e)
    {
        await Navigation.PopModalAsync();
    }
}
<StackLayout Padding="5">
    <Button x:Name="ToCommonPage" Text="На обычную страницу" HorizontalOptions="Center" VerticalOptions="CenterAndExpand" Clicked="ToCommonPage_Clicked" />
    <Button x:Name="ToModalPage" Text="На модальную страницу" HorizontalOptions="Center" VerticalOptions="CenterAndExpand" Clicked="ToModalPage_Clicked" />
</StackLayout>
private async void ToCommonPage_Clicked(object sender, EventArgs e)
{
    await Navigation.PushAsync(new CommonPage());
}
private async void ToModalPage_Clicked(object sender, EventArgs e)
{
    await Navigation.PushModalAsync(new ModalPage());
}
```

## Передача данных при навигации



