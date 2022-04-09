# NuGet

## Настройка библиотеки

Пример настроек файла библиотеки общих элементов на dotnet 6:

```csharp
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <Version>0.0.1</Version>
	<PackageReleaseNotes>
		Первая версия
	</PackageReleaseNotes>
  </PropertyGroup>

  <PropertyGroup>
	<TargetFramework>net6.0</TargetFramework>
	<ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>
	
  <PropertyGroup>
    <Nullable>enable</Nullable>
    <Authors>Rassakhatskiy Andrey</Authors>
    <Company>Kanadeiar</Company>
    <Description>Основная библиотека 📚 прикладных инструментов, алгоритмов и инструкций.</Description>
    <Copyright>Kanadeiar</Copyright>
    <PackageProjectUrl>https://github.com/kanadeiar/Kanadeiar.Core</PackageProjectUrl>
    <RepositoryUrl>https://github.com/kanadeiar/Kanadeiar.Core</RepositoryUrl>
    <PackageIcon>42916202.png</PackageIcon>
    <PackageReadmeFile>README.md</PackageReadmeFile>
    <PackageTags>dotnet, tools, algorithms, instructions</PackageTags>
    <SignAssembly>True</SignAssembly>
    <DelaySign>False</DelaySign>
    <AssemblyOriginatorKeyFile>Kanadeiar.snk</AssemblyOriginatorKeyFile>
	<SymbolPackageFormat>snupkg</SymbolPackageFormat>
	<PackageLicenseExpression>GPL-3.0-or-later</PackageLicenseExpression>
	<Language>C#</Language>
	<Title>Kanadeiar.Core</Title>
  </PropertyGroup>

  <ItemGroup>
    <None Include="..\README.md">
      <Pack>True</Pack>
      <PackagePath>\</PackagePath>
    </None>
    <None Include="C:\Users\Admin\Pictures\42916202.png">
      <Pack>True</Pack>
      <PackagePath>\</PackagePath>
    </None>
  </ItemGroup>

</Project>
```

Команда создания пакета:

```csharp
dotnet pack -c Release -o ..\Release
```

## Автоматизация

Конфиг тестирования:

```csharp
name: Tests

on:
  [ push, pull_request ]

jobs:
  test:
    name: Test on dotnet 6
    runs-on: ubuntu-latest

    steps:
    - name: Checkout
      uses: actions/checkout@v3
    
    - name: Setup .NET 6
      uses: actions/setup-dotnet@v2
      with:
        dotnet-version: 6.0.x
        
    - name: Build
      run: |
        dotnet build Kanadeiar.Core  -c debug
        dotnet build Tests/Kanadeiar.Core.Tests -c debug
      
    - name: Test
      run: dotnet test Tests/Kanadeiar.Core.Tests --no-build -c debug -v normal
```

Конфиг публикации в nuget:

```csharp
name: Publish NuGet Package

on:
  push:
    branches: [ main ]

jobs:
  build:
    name: Update package
    runs-on: ubuntu-latest

    steps:
    - name: Checkout
      uses: actions/checkout@v3
      
    - name: Setup .NET 6
      uses: actions/setup-dotnet@v2
      with:
        dotnet-version: 6.0.x
      
    - name: Build
      run: |
        dotnet build Kanadeiar.Core  -c Release
        dotnet build Tests/Kanadeiar.Core.Tests -c Release
      
    - name: Test
      run: dotnet test Tests/Kanadeiar.Core.Tests --no-build -c Release -v normal
      
    - name: Pack
      run: dotnet pack Kanadeiar.Core -c Release --no-build -v normal
      
    - name: Publish
      run: dotnet nuget push "**/*.nupkg" -k ${{ secrets.NuGetApiKey }} -n --skip-duplicate -s https://api.nuget.org/v3/index.json
```