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
USE shop;
INSERT INTO Items (Name, ItemID) VALUES ('Тест номер 1', 101), ('Тест номер 2', 102);
```

Ограничение на выборку данных из таблицы:
```sql
SELECT * FROM Items;
SELECT * FROM Items WHERE (ItemID>5) AND (ItemID<200);
SELECT * FROM Items WHERE NOT (ItemID=100);
SELECT * FROM Items WHERE ItemID IS NOT NULL;
```
Выборка части информации:
```sql
SELECT Name FROM Items;
SELECT DISTINCT Name FROM Items; --уникальные имена выводить только
SELECT * FROM Items ORDER BY Name ASC; --прямой порядок
SELECT * FROM Items ORDER BY Name DESC; --обратный порядок
SELECT * FROM Items WHERE ItemID != 0 ORDER BY Name DESC; --обратный порядок
SELECT TOP 2 * FROM Items; --только 2 записи
SELECT TOP 2 * FROM Items WHERE ItemID != 0; --только 2 записи с условием
```

Изменение записей в таблице:
```sql
UPDATE Items SET name='Изменное' WHERE ItemID=102
UPDATE Items SET ItemID=105 WHERE ItemID=102 OR ItemID=101
UPDATE Items SET ItemID=105 WHERE ItemID IN (100, 101);
```

Удаление записей из таблицы:
```sql
DELETE FROM Items WHERE ItemID > 100;
```

## Согласованность данных

К примеру таблица содержащая значения из других таблиц:
```sql
CREATE TABLE Product
(
Id INT IDENTITY(1,1) NOT NULL,
BrandID INT NOT NULL,
GoodID INT NOT NULL,
CategoryID INT NOT NULL,
Price DECIMAL(10,2) NOT NULL,
CONSTRAINT PK_IdProduct PRIMARY KEY (Id)
)
//добавление значений в нее
INSERT INTO Product (BrandID, GoodID, CategoryID, Price)
VALUES (1, 1, 1, 8999.9);
```
Внешние ограничения (или ключи):
```sql
ALTER TABLE Product 
ADD CONSTRAINT FK_ProductToBrand FOREIGN KEY (Id)
	REFERENCES Brand (Id);
ALTER TABLE Product 
ADD CONSTRAINT FK_ProductToGood FOREIGN KEY (GoodId)
	REFERENCES Good (Id);
```
Таблица с отношениями "многие ко многим":
```sql
CREATE TABLE "Order" --таблица с заказами
(
Id INT IDENTITY(1,1) NOT NULL,
UserName VARCHAR(128) NOT NULL,
Phone VARCHAR(32) NOT NULL,
Datetime DATETIME NOT NULL,
CONSTRAINT FK_IdOrder PRIMARY KEY(Id)
)
//таблица с продуктами каждого каказа, промежуточная таблица, со составным ключом
CREATE TABLE OrderProducts
(
OrderID INT NOT NULL,
ProductID INT NOT NULL,
[Count] INT,
CONSTRAINT PK_OrPrID PRIMARY KEY (OrderID, ProductID), --уникальность первых двух полей
CONSTRAINT FK_OrPrOrderID FOREIGN KEY (OrderID) --внешний ключ 1
	REFERENCES [Order] (Id),
CONSTRAINT FK_OrPrProductID FOREIGN KEY (ProductID) --внешний ключ 2
	REFERENCES [Product] (Id)
)
//добавление строк
INSERT INTO OrderProducts (OrderID, ProductID, [Count])
VALUES (1, 1, 1), (1, 2, 3);
```

## Объединение данных





