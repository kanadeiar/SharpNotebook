# RabbitMQ

## Общие консольные команды

Затягивание докер-образа раббита из репозитория:

```csharp
docker run -it --rm --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:3.9-management
```