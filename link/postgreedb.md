# PostgreeDB

## Общие консольные команды

Затягивание докер-образа раббита из репозитория:

```csharp
docker run --name postgres -p 5432:5432 -e POSTGRES_PASSWORD=password -d postgres
```