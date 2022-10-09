# Команды dotnet

## Общие команды

Создание решения:

```csharp
dotnet new sln -n SimpleSolution -o .\SimpleSolutionDir
```

Создание консольного приложения:

```csharp
dotnet new console -lang c# -n SimpleConsoleApp -o .\SimpleSolutionDir\
SimpleConsoleApp -f net5.0
```

Добавление приложения в решение:

```csharp
dotnet sln .\SimpleSolutionDir\SimpleSolution.sln add .\SimpleSolutionDir\
SimpleConsoleApp
```


