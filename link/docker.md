# Docker

## Действия с репозиториями

Поиск репозитория:

```csharp
docker search nginx
```

Поиск репозитория:

```csharp
docker pull nginx
```

Push (загрузка в реестр) образа:

```csharp
docker push eon01/nginx
```

## Действия с контейнерами

Создание контейнера:

```csharp
docker create -t -i eon01/test --name test
```
Запуск контейнера:

```csharp
docker run -it --name test -d eon01/test
```

Переименование контейнера:

```csharp
docker 
```

Удаление контейнера:

```csharp
docker 
```