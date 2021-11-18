# Интернет Xamarin.Forms

## Инфраструктура

<PackageReference Include="System.Net.Http.Json" Version="6.0.0" />
<PackageReference Include="System.Text.Json" Version="6.0.0" />

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
App.xaml.cs
```csharp
services.AddScoped(sp => new HttpClient { BaseAddress = new Uri("https://kanadeiarapi.azurewebsites.net") });
services.AddScoped<FriendClient>();
```

MainPageViewModel
```csharp
private readonly FriendClient _friendClient;
...
public ObservableCollection<Friend> Friends { get; set; } = new ObservableCollection<Friend>();
...
public MainPageViewModel(FriendClient friendClient)
{
    _friendClient = friendClient;

    Task.Run(UpdateFriends);
}
...
public async Task UpdateFriends()
{
    var friends = await _friendClient.GetAll();
    Friends.Clear();
    foreach (var friend in friends)
        Friends.Add(friend);
}
```

