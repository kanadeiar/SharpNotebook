# WPF Core MVVM

Пример реализации MVVM на Core.

1. Установить NuGet пакет Microsoft.Extensions.Hosting, включающий в себя всю инфраструктуру MVVM.

2. Код:

Базовый класс вьюмодели "ViewModel":
```csharp
namespace WpfApp1.ViewModels.Base
{
    abstract class ViewModel : INotifyPropertyChanged
    {
        public event PropertyChangedEventHandler PropertyChanged;
        protected virtual void OnPropertyChanged([CallerMemberName] string propertyName = null)
        {
            PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
        }
        protected virtual void Set<T>(ref T field, T value, [CallerMemberName] string propertyName = null)
        {
            if (Equals(field, value)) return;
            field = value;
            OnPropertyChanged(propertyName);
        }
    }
}
```

Класс вьюмодели главного окна "MainWindowViewModel":
```csharp
namespace WpfApp2.ViewModels
{
    class MainWindowViewModel : ViewModel
    {
        private string _title = "Главное окно программы";
        public string Title
        {
            get => _title;
            set => Set(ref _title, value);
        }
        public MainWindowViewModel()
        {
        }
    }
}
```

Добавление в файл приложения хостинга сервисов "App.xaml.cs":
```csharp
namespace WpfApp2
{
    /// <summary>
    /// Interaction logic for App.xaml
    /// </summary>
    public partial class App : Application
    {
        private static IHost __hosting;

        public static IHost __Hosting
        {
            get
            {
                if (__hosting != null) return __hosting;
                var hostBuilder = Host.CreateDefaultBuilder(Environment.GetCommandLineArgs());
                hostBuilder.ConfigureServices(ConfigureServices);
                return __hosting = hostBuilder.Build();
            }
        }
        public static IServiceProvider Services => __Hosting.Services;
        private static void ConfigureServices(HostBuilderContext host, IServiceCollection services)
        {
            services.AddSingleton<IDialogService, WindowDialog>();
            services.AddSingleton<MainWindowViewModel>();
        }
    }
    #region Тестовый сервис показа сообщения
    interface IDialogService
    {
        void ShowInfo(string msg);
    }
    class WindowDialog : IDialogService
    {
        public void ShowInfo(string msg) => MessageBox.Show(msg);
    }
    #endregion
}
```

Локатор вьюмоделей "ViewModelLocator", где регистрируются все вьюмодели:
```csharp
namespace WpfApp1.ViewModels
{
    class ViewModelLocator
    {
        public MainWindowViewModel MainWindowModel => App.Services.GetRequiredService<MainWindowViewModel>();
    }
}
```

Добавление локатора в ресурсы приложения:
```csharp
...
             xmlns:vm="clr-namespace:WpfApp2.ViewModels"
...
            <vm:ViewModelLocator x:Key="Locator"/>
...
```

3. Перестроить приложение.

4. Можно использовать вьмодель из локатора "MainWindow.xaml":
```csharp
namespace WpfApp1.ViewModels
{
    class ViewModelLocator
    {
        public MainWindowViewModel MainWindowModel => App.Services.GetRequiredService<MainWindowViewModel>();
    }
}
```

5. Добавление базовой команда и лямбда-команды:


