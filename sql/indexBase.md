# Базовые команды MSSQL

## Создание таблицы в базе данных

Команда создания таблицы:
```sql
CREATE TABLE Items
(
IdItem INT IDENTITY(1,1) NOT NULL,
Name VARCHAR(64) NOT NULL,
ItemID INT NOT NULL,
CONSTRAINT PK_IdItems PRIMARY KEY (IdItem)
)
GO
```
Команда изменения таблицы:
```sql
ALTER TABLE Items
ADD Add_name VARCHAR(200) NULL;
```
Команда удаления таблицы:
```sql
DROP TABLE Items;
```

Добавление записей в таблицу:
```sql
INSERT INTO Items (Name, ItemID) VALUES ('101.Давление на входе', 100)
```