# Интернет Xamarin.Forms

## Инфраструктура

<PackageReference Include="Microsoft.Extensions.DependencyInjection" Version="6.0.0" />
<PackageReference Include="System.Net.Http.Json" Version="6.0.0" />
<PackageReference Include="System.Text.Json" Version="6.0.0" />
<PackageReference Include="Xamarin.Forms" Version="5.0.0.2244" />
<PackageReference Include="Xamarin.Essentials" Version="1.7.0" />

Base.FriendClient
```csharp
public abstract class BaseClient
{
    protected readonly HttpClient Client;
    protected readonly string Address;
    public BaseClient(HttpClient client, string address)
    {
        Client = client;
        Address = address;
    }
    /// <summary> Асинхронное получение данных с веб апи сервера </summary>
    /// <typeparam name="T">тип данных</typeparam>
    /// <param name="url">конечная точка</param>
    /// <param name="cancel">токен отмены</param>
    /// <returns>данные</returns>
    protected async Task<T> GetAsync<T>(string url, CancellationToken cancel = default)
    {
        var response = await Client
            .GetAsync(url, cancel).ConfigureAwait(false);
        if (response.StatusCode == HttpStatusCode.NoContent)
            return default;
        return await response
            .EnsureSuccessStatusCode()
            .Content.ReadFromJsonAsync<T>(cancellationToken: cancel);
    }
    /// <summary> Асинхронное добавление данных в веб апи сервер </summary>
    /// <typeparam name="T">тип данных</typeparam>
    /// <param name="url">конечная точка</param>
    /// <param name="item">данные</param>
    /// <param name="cancel">токен отмены</param>
    /// <returns>статус добавления</returns>
    protected async Task<HttpResponseMessage> PostAsync<T>(string url, T item, CancellationToken cancel = default)
    {
        var response = await Client
            .PostAsJsonAsync(url, item, cancel).ConfigureAwait(false);
        return response.EnsureSuccessStatusCode();
    }
    /// <summary> Асинхронное обновление данных в веб апи сервере </summary>
    /// <typeparam name="T">тип данных</typeparam>
    /// <param name="url">конечная точка</param>
    /// <param name="item">данные</param>
    /// <param name="cancel">токен отмены</param>
    /// <returns>результат обновления</returns>
    protected async Task<HttpResponseMessage> PutAsync<T>(string url, T item, CancellationToken cancel = default)
    {
        var response = await Client
            .PutAsJsonAsync(url, item, cancel).ConfigureAwait(false);
        return response.EnsureSuccessStatusCode();
    }
    /// <summary> Асинхронное удаление данных из веб апи сервера </summary>
    /// <param name="url">конечная точка</param>
    /// <param name="cancel">токен отмены</param>
    /// <returns>результат обновления</returns>
    protected async Task<HttpResponseMessage> DeleteAsync(string url, CancellationToken cancel = default)
    {
        var response = await Client
            .DeleteAsync(url, cancel).ConfigureAwait(false);
        return response;
    }
}
```

FriendClient
```csharp
public class FriendClient : BaseClient
{
    public FriendClient(HttpClient client) : base(client, "/api/friend") { }

    public async Task<IEnumerable<Friend>> GetAll()
    {
        return await GetAsync<IEnumerable<Friend>>(Address).ConfigureAwait(false);
    }
    public async Task<Friend> Get(int id)
    {
        return await GetAsync<Friend>($"{Address}/{id}").ConfigureAwait(false);
    }
    public async Task<int> Add(Friend friend)
    {
        var response = await PostAsync(Address, friend).ConfigureAwait(false);
        return await response.Content.ReadFromJsonAsync<int>();
    }
    public async Task Update(Friend friend)
    {
        await PutAsync(Address, friend);
    }
    public async Task<bool> Delete(int id)
    {
        var result = (await DeleteAsync($"{Address}/{id}")).IsSuccessStatusCode;
        return result;
    }
}
```

App.xaml
```csharp
<vm:ViewModelLocator x:Key="Locator" />
```
App.xaml.cs
```csharp
private static IServiceProvider _Services;
private static IServiceCollection GetServices()
{
    var services = new ServiceCollection();
    InitializeServices(services);
    return services;
}
/// <summary> Сервисы приложения </summary>
public static IServiceProvider Services => _Services ??= GetServices().BuildServiceProvider();
private static void InitializeServices(IServiceCollection services)
{
    services.AddScoped(sp => new HttpClient { BaseAddress = new Uri("https://kanadeiarapi.azurewebsites.net") });

    services.AddScoped<FriendClient>();

    services.AddTransient<MainPageViewModel>();
}
public App()
{
    InitializeComponent();
    MainPage = new NavigationPage( new MainPage() );
}
```

MainPageViewModel
```csharp
private readonly FriendClient _friendClient;
...
public ObservableCollection<Friend> Friends { get; set; } = new ObservableCollection<Friend>();
private Friend _selectedFriend;
/// <summary> Выбранный друг </summary>
public Friend SelectedFriend
{
    get => _selectedFriend;
    set => Set( ref _selectedFriend, value );
}
...
public MainPageViewModel(FriendClient friendClient)
{
    _friendClient = friendClient;
}
...
private ICommand _UpdateFriendsCommand;
/// <summary> Обновить </summary>
public ICommand UpdateFriendsCommand => _UpdateFriendsCommand ??= new Command(OnUpdateFriendsCommandExecuted);
private async void OnUpdateFriendsCommandExecuted(object p)
{
    RefreshFriends = true;
    await UpdateFriends();
    RefreshFriends = false;
}
...
public async Task UpdateFriends()
{
    var friends = await _friendClient.GetAll();
    Friends.Clear();
    foreach (var friend in friends)
        Friends.Add(friend);
    SelectedFriend = null;
}
```
MainPage.xaml
```xml
...
            Title="{Binding Title}"             
            BindingContext="{Binding MainPageViewModel, Source={StaticResource Locator}}">
<StackLayout>
    <Grid>
        <Grid.ColumnDefinitions>
            <ColumnDefinition/>
            <ColumnDefinition/>
            <ColumnDefinition/>
        </Grid.ColumnDefinitions>
        <Button Grid.Column="0" Text="Создать" BackgroundColor="LawnGreen" Margin="4,4,4,0" Padding="-10" FontSize="Subtitle" HeightRequest="80"
            Command="{Binding AddFriendsCommand}" />
        <Button Grid.Column="1" Text="Изменить" BackgroundColor="Gold" Margin="4,4,4,0" Padding="-10" FontSize="Subtitle" HeightRequest="80"
            Command="{Binding EditFriendsCommand}" CommandParameter="{Binding SelectedFriend}" />
        <Button Grid.Column="2" Text="Удалить" BackgroundColor="IndianRed" Margin="4,4,4,0" Padding="-10" FontSize="Subtitle" HeightRequest="80"
            Command="{Binding DeleteFriendsCommand}" CommandParameter="{Binding SelectedFriend}" />
    </Grid>
    <ListView x:Name="ListViewFriends" HasUnevenRows="True" IsPullToRefreshEnabled="True"
            ItemsSource="{Binding Friends}" 
            SelectedItem="{Binding SelectedFriend}"  
            RefreshCommand="{Binding UpdateFriendsCommand}" 
            IsRefreshing="{Binding RefreshFriends}">
        <ListView.ItemTemplate>
            <DataTemplate x:DataType="m:Friend">
                <ViewCell>
                    <ViewCell.View>
                        <StackLayout Padding="10,0">
                            <Label Text="{Binding Name}" FontSize="Subtitle" />
                            <Label Text="{Binding Email}" FontSize="Body" />
                            <Label Text="{Binding Phone}" FontSize="Body" />
                        </StackLayout>
                    </ViewCell.View>
                </ViewCell>
            </DataTemplate>
        </ListView.ItemTemplate>
    </ListView>
</StackLayout> 
```
MainPage.xaml.cs
```csharp
public MainPageViewModel ViewModel { get; set; }

public MainPage()
{
    InitializeComponent();
    ViewModel = App.Services.GetRequiredService<MainPageViewModel>();
}
```
EditFriendPage.xaml
```xml
<StackLayout x:Name="StackLayoutEditFriend" x:DataType="m:Friend">
    <Label Text="Имя" />
    <Entry Text="{Binding Name}" />
    <Label Text="Почта" />
    <Entry Text="{Binding Email}" />
    <Label Text="Телефон" />
    <Entry Text="{Binding Phone}" />
    <Grid>
        <Grid.ColumnDefinitions>
            <ColumnDefinition/>
            <ColumnDefinition/>
        </Grid.ColumnDefinitions>
        <Button Grid.Column="0" Text="OK" BackgroundColor="LawnGreen" Margin="4" Padding="-10" FontSize="Subtitle" HeightRequest="80"
                Clicked="ButtonOK_Clicked"/>
        <Button Grid.Column="1" Text="Отмена" BackgroundColor="Gold" Margin="4" Padding="-10" FontSize="Subtitle" HeightRequest="80"
                Clicked="ButtonCancel_Clicked"/>
    </Grid>
</StackLayout>
```
EditFriendPage.xaml.cs
```csharp
public partial class EditFriendPage : ContentPage
{
    public MainPageViewModel ViewModel { get; set; }
    public Friend Friend { get; set; }
    private Action<Friend> _action;

    public EditFriendPage(string title, Friend friend, Action<Friend> action)
    {
        InitializeComponent();
        ViewModel = App.Services.GetRequiredService<MainPageViewModel>();
        Title = title;
        StackLayoutEditFriend.BindingContext = Friend = friend;
        _action = action;
    }

    private async void ButtonOK_Clicked(object sender, EventArgs e)
    {
        if (string.IsNullOrEmpty(Friend.Name) || string.IsNullOrEmpty(Friend.Email) || string.IsNullOrEmpty(Friend.Phone))
        {
            await DisplayAlert("Ошибка ввода", "Необходимо ввести данные во все поля", "OK");
            return;
        }

        await Navigation.PopAsync();

        _action.Invoke(Friend);
    }

    private async void ButtonCancel_Clicked(object sender, EventArgs e)
    {
        await Navigation.PopAsync();
    }
}
```

