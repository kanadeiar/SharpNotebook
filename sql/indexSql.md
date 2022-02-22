# Основы базы данных MSSQL

## Нормальные формы 

Первая нормальная форма (1NF) 

Основные критерии: 

* Все строки должны быть различными. 

* Все элементы внутри ячеек должны быть атомарными (не списками). Другими словами, элемент является атомарным, если его нельзя разделить на части, которые могут использовать в таблице независимо друг от друга.

Методы приведения к 1NF: 

* Устраните повторяющиеся группы в отдельных таблицах (одинаковые строки). 

* Создайте отдельную таблицу для каждого набора связанных данных. 

* Идентифицируйте каждый набор связанных данных с помощью первичного ключа (добавить уникальный id для каждой строки)

Вторая нормальная форма (2NF) 

Основные критерии: 

* Таблица должна находиться в первой нормальной форме. 

* Любое её поле, не входящее в состав первичного ключа, функционально полно зависит от первичного ключа. 

Ваша таблица приведена к первой нормальной форме и у нее установлен уникальный id для каждой строки, то она находится и во второй нормальной форме.

Методы приведения к 2NF: 

* Создайте отдельные таблицы для наборов значений, относящихся к нескольким записям 

* Свяжите эти таблицы с помощью внешнего ключа 


Третья нормальная форма (3NF) 

Основные критерии: 

* Таблица находится во второй нормальной форме. 

* Любой её не ключевой атрибут функционально зависит только от первичного ключа.


Проще говоря, второе правило требует выносить все не ключевые поля, содержимое которых может относиться к нескольким записям таблицы в отдельные таблицы.

Методы приведения к 3NF:

* Удаление полей не зависящих от ключа

## Индексы базы данных

Что такое индекс? Если говорить простыми словами, то это способ отсортировать данные по определенной колонке. Когда список отсортирован, намного проще производить поиск необходимых данных. 

Понимание того, как хранятся данные – является основой понимание того, как получить к ним доступ. Для начала нам нужно разобраться с таким понятием как куча – это коллекция страниц данных, содержащих строки для таблицы:

* Каждая страница данных содержит 8 килобайт информации. Группа и 8-и рядом стоящих страниц называется пространством.
* Строки данных не хранятся в каком-либо определенном порядке, и нет определенного порядка для последовательности страниц.
* Страницы данных не связаны в связанные списки.
* Когда строка вставляется в страницу и страница переполнена, страница разделяется.

Сервер SQL получает доступ к данным одним из следующих способов:

* Сканирует все страницы таблицы – сканирование таблицы. Когда SQL Server выполняет сканирование таблицы он:
- Начинает с начала таблицы;
- Сканирует от страницы к странице через все строки таблицы;
- Выделяет строку, которая соответствует запросу.
* Используя индексы. Когда SQL Server использует индексы, он:
- Пересекает структуру дерева индексов для поиска строк, соответствующих запросу;
- Выделяет только необходимые строки, соответствующие критериям запроса.

Первым делом, SQL Server определяет, какие индексы существуют. Оптимизатор запроса (компонент, предназначенный для генерирования оптимального плана для запроса) определяет что использовать – сканировать таблицу или индексы. Индексы более предпочтительны.

Когда вы рассматриваете, нужно ли создавать индексы, рассчитайте два фактора, для гарантирования, что индексы будут более эффективны, чем сканирование таблицы: природа данных и природа запросов к таблице.

Индексы ускоряют доступ к данным. Для примера, без индекса, без индексы вам понадобится перелистать постранично всю книгу для определения содержания. По содержанию легче найти интересующую информацию. 

Сервер SQL использует индексы для указания на расположение строки в странице данных вместо просматривания всех страниц таблицы. Рассматривайте следующие факты и рекомендации об индексах:

* Индексы обычно увеличивают скорость выполнения запросов связанных таблиц и выполнение сортировки и группировки;
* Индексы принуждают делать строки уникальными, если включена уникальность.
* Индексы создаются в порядке возрастания или уменьшения.

Индексы достаточно полезны, но они занимают место на диске и берут на себя дополнительные накладные расходы и расходы на эксплуатацию. Индексы могут создать и проблемы:

* когда вы изменяете данные в индексной колонке, сервер SQL обновляет связанные индексы.
* накладные расходы на поддержку индексов требуют времени и ресурсов. Поэтому не создавайте индексы, которые не будете часто использовать.
* индексы на колонки, содержащие большое количество доблирующих данных могут иметь несколько преимуществ.

## Связанные таблицы

Ортогональное объединение по методу MSSQL, т.е. без указания связи:
```sql
SELECT * 
FROM tbPeoples CROSS JOIN tbPosition
```
Внутреннее объединение (эквивалентно знаку равенства) по методу MSSQL описывается следующим образом:
```sql
SELECT * 
FROM tbPeoples pl INNER JOIN tbPosition ps 
     ON pl.idPosition=ps.idPosition
```
Объединение двух таблиц в одно целое. Запрос:
```sql
SELECT * 
FROM tbPeoples pl LEFT OUTER JOIN tbPhoneNumbers pn
     ON pl.idPeoples=pn.idPeoples
```
Чтобы получить правое объединение, необходимо просто поменять перечисление таблиц местами:
```sql
SELECT * 
FROM tbPhoneNumbers pn RIGHT OUTER JOIN tbPeoples pl 
     ON pl.idPeoples=pn.idPeoples
```
Меняется местами перечисление таблиц, а вот порядок указания связанных полей не имеет особого значения и здесь ничего не меняется. 

## Хранимые процедуры

Хранимые процедуры – это именованный набор операторов Transact-SQL хранящийся на сервере. Хранимые процедуры – это метод выполнения повторяющихся задач и при этом обладают большими возможностями, чем объекты просмотра. 

Встроенные процедуры:

* системные хранимые процедуры;
* локальные хранимые процедуры;
* временные хранимые процедуры;
* расширенные встроенные процедуры (содержат в имени префикс xp_).

Процедура - это блок из одной или более команд. Это может быть не просто один запрос, а целая программа, с собственной логикой (операторы IF), циклами. Процедура может принимать заранее определенные переменные и использовать их в своих расчетах, благодаря чему, результат работы процедуры может быть динамическим, и будет зависеть от определенных условий и/или состояния получаемых переменных. 

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

Объединение данных из нескольких таблиц. 

Объединение путем пересечения нескольких множеств. В результирующий запрос попаюают только те строки, которые являются пересечением нескольких таблиц, которые существуют в обоих таблицах:
```sql
//полные таблицы
SELECT * FROM Product
INNER JOIN Category on Product.CategoryID = Category.Id
//выборочные
SELECT Product.BrandID, Product.GoodID, Category.Category, Product.Price, Category.Discount FROM Product
INNER JOIN Category on Product.CategoryID = Category.Id
//более компактно
SELECT tp.BrandID, tp.GoodID, tc.Category, tp.Price, tc.Discount FROM Product tp
INNER JOIN Category tc on tp.CategoryID = tc.Id
//несколько таблиц
SELECT * FROM Product
	INNER JOIN Category ON Product.CategoryID=Category.Id
	INNER JOIN Brand ON Product.BrandID=Brand.Id
	INNER JOIN Good ON Product.GoodID=Good.Id
//выборака полей из нескольких таблиц
SELECT tp.Id, Brand.Brand, Category.Category, Good.Good, tp.Price FROM Product tp
	INNER JOIN Category ON tp.CategoryID=Category.Id
	INNER JOIN Brand ON tp.BrandID=Brand.Id
	INNER JOIN Good ON tp.GoodID=Good.Id
//еще пример - вторая таблица для отношения многие ко многим
SELECT * FROM "Order"
	INNER JOIN OrderProducts ON OrderProducts.OrderID="Order".Id
	INNER JOIN Product ON OrderProducts.ProductID=Product.Id
```

Объединение левое - в результирующий запрос попаюают все строки, которые есть в множестве ловом и в пересечении с правым множеством. Недостающие строки заполняются значениями NULL:
```sql
//левое объединение
SELECT * FROM Category
	LEFT JOIN Product ON Product.CategoryID=Category.Id
//из левой таблицы знаяения которым нет сопоставления в правой таблице
SELECT * FROM Category
	LEFT JOIN Product ON Product.CategoryID=Category.Id
	WHERE Product.Id IS NULL;
//только инфа из таблицы левой
SELECT Category.* FROM Category
	LEFT JOIN Product ON Product.CategoryID=Category.Id
	WHERE Product.Id IS NULL;
//правое объединение
SELECT * FROM Category
	RIGHT JOIN Product ON Product.CategoryID=Category.Id
	WHERE Product.Id IS NULL;
```

Объединение полное - полное всех возможных вариатов строк.
```sql
//полное объединение
SELECT * FROM "Order"
	FULL OUTER JOIN OrderProducts ON OrderProducts.OrderID="Order".Id
	FULL OUTER JOIN Product ON OrderProducts.ProductID=Product.Id
//объединение через UNION
SELECT * FROM Good WHERE Id=2
UNION
SELECT * FROM Good WHERE Id=1;
```

## Агрегация (группировка) данных

Примеры простой агрегации:
```sql
//количество
SELECT count(*) as Количество FROM Product
//количество с условием
SELECT count(*) as Количество FROM Product WHERE Price < 1000
//сумма всех значений столбца, минимальное и максимальное
SELECT sum(Price) as Summ, max(Price) as Minim, min(Price) as Minim FROM Product
//подсчет суммы всех заказов для некоторых людей
SELECT sum(Count * Price) as Сумма FROM "Order"
	INNER JOIN OrderProducts ON "Order".Id=OrderProducts.OrderID
	INNER JOIN Product ON OrderProducts.ProductID=Product.Id
	WHERE "Order".UserName LIKE 'Гоша'
```
Группировка - в последнем разделе нужно указать по какому столбцу нужно группировать, а в первом - обязательно название этого столбца нужно указать и функцию агрегации с какими либо другими столбцами.
```sql
//группировка по имени пользователя
SELECT "Order".UserName, sum(Count * Price) as Сумма FROM "Order"
	INNER JOIN OrderProducts ON "Order".Id=OrderProducts.OrderID
	INNER JOIN Product ON OrderProducts.ProductID=Product.Id
	GROUP BY "Order".UserName
//максимальное значение + количество
SELECT "Order".UserName, max(Price) as Сумма, count(*) as Количество FROM "Order"
	INNER JOIN OrderProducts ON "Order".Id=OrderProducts.OrderID
	INNER JOIN Product ON OrderProducts.ProductID=Product.Id
	GROUP BY "Order".UserName
//запросы с условиями
SELECT "Order".UserName, sum(Price * Count) as Сумма FROM "Order"
	INNER JOIN OrderProducts ON "Order".Id=OrderProducts.OrderID
	INNER JOIN Product ON OrderProducts.ProductID=Product.Id
	WHERE "Order".UserName LIKE '%а%'
	GROUP BY "Order".UserName
```
Условия в аггрегирующих функциях - HAVING:
```sql
SELECT "Order".UserName, count(Count) as count_ord FROM "Order"
	INNER JOIN OrderProducts ON "Order".Id=OrderProducts.OrderID
	INNER JOIN Product ON OrderProducts.ProductID=Product.Id
	GROUP BY "Order".UserName
	HAVING count(Count) >= 3
```


