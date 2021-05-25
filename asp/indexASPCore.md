# Веб ASP.NET Core 5 Введение

В папке wwwroot хранится содержимое клиентской стороны, такое как стили CSS, сценарии JavaScript. 

В ASP.NET Core есть только один тип, от которого следует наследовать контроллеры - Controller. Действия возвращают IActionResult или Task<IActionResult>.

Все основано на внедрении зависимостей, не только Startup получает все службы конфигурации и промежуточное ПО через внедрение зависимостей, сюда также следует включать и свои крассы, чтоб их внедрять в другие части приложения.

Контейнер конфигурируется в методе ConfigureServices() класса Startup с применением экземляра IServiceCollection, сам он тоже внедряется. 

Вариант существования    | Описание
-------------------------|-------------------------
Transient                | Временный, создается каждый раз, когда в нем возникает необходимость
Scoped                   | С заданной областью, создается один раз для каждого запроса, рекомендуется для DbContext
Singleton                | Один раз создается при первом запуске и потом многократно используется
Instance                 | Подобно Singleton, но создается один раз при обращении к элементу в первом запросе

Файл настроек приложения "appsettings.Development.json" со строкой подключения:
```csharp
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information"
    }
  },
  "ConnectionStrings": {
    "Students": "Data Source=(localdb)\\MSSQLLocalDB;Initial Catalog=StudentsDev.DB; Integrated security=True; MultipleActiveResultSets=True; App=EntityFramework;"
  }
}
```

## Дескрипторы

Для элементов ввода в приложениях ASP.NET Core рекомендуется использовать вспомогательные функции дескрипторов.

Дескриптор       | Метод HTML | Атрибут | Описание
-----------------|------------|---------|----------
form             | Html.BeginForm() HTML.BeginRouteForm() HTML.AntiForgeryToken() | asp-route | Для именованных маршрутов
|  |  | asp-antiforgery | Маркер противодействия подделки (true по умолчанию)
|  |  | asp-area        | Устанавливает имя области
|  |  | asp-controller  | Устанавливает контроллер
|  |  | asp-action      | Устанавливает действие
|  |  | asp-route-<параметр> | Добавляет параметр к маршруту (asp-route-id="1")
|  |  | asp-all-route-data | словать с дополнительными значениями для маршрута
a                | Html.ActionLink() | asp-route | Для именованных маршрутов (без контроллеров и действий)
|  |  | asp-area        | Устанавливает имя области
|  |  | asp-controller  | Определяет контроллер
|  |  | asp-action      | Устанавливает действие
|  |  | asp-protocol    | Определяет протокол HTTP или HTTPS
|  |  | asp-fragment    | Определяет фрагмент URL
|  |  | asp-host        | Устанавливает имя хоста
|  |  | asp-route-<параметр> | Добавляет параметр к маршруту (asp-route-id="1")
|  |  | asp-all-route-data | словать с дополнительными значениями для маршрута





