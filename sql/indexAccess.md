# База данных Access

## Таблицы

Работать с базой данных можно без окон - напрямую запросами SQL.

Создание таблицы:
```sql
CREATE TABLE [Пример]
(
[id] INTEGER NOT NULL PRIMARY KEY,
[Страна] VARCHAR(50) NOT NULL,
[Транспорт] VARCHAR(50) NOT NULL,
[Стоимость] MONEY NOT NULL
);
```
Добавление записи в таблицу:
```sql
INSERT INTO Пример
VALUES (1, 'Россия', 'автобус', 1200);
```
Выборка данных из таблицы:
```sql
SELECT *
FROM Пример;
```
Еще:
```sql
SELECT [Страна], [Транспорт]
FROM Пример;
```
Еще:
```sql
SELECT *
FROM Пример
WHERE [Страна] = 'Россия';
```
Еще:
```sql
SELECT *
FROM Пример
ORDER BY [Стоимость] DESC;
```
Получение результата - одно число:
```sql
SELECT MIN(Стоимость)
FROM Пример;
```
Выборка по вложенному запросу:
```sql
SELECT * FROM [Пример] WHERE [Стоимость] = 
(SELECT MIN([Стоимость]) FROM [Пример] WHERE [Транспорт]='самолет')
and ([Транспорт]='самолет')
```
Обновление данных в таблице:
```sql
UPDATE [Пример] SET [Стоимость] = [Стоимость] * 1.1
WHERE [Транспорт] = 'самолет'; 
```
Удаление данных из таблицы:
```sql
DELETE FROM [Пример]
WHERE [Транспорт] = 'самолет'; 
```
Удаление таблицы:
```sql
DROP TABLE [Пример]
```
Изменение таблицы:
```sql
ALTER TABLE [Пример]
(
[Еще] INTEGER NOT NULL
);
```

## Реляционка

Создание таблиц:
```sql
CREATE TABLE [Заказы]
(
[Номер] INTEGER NOT NULL PRIMARY KEY,
[Дата] DATETIME NOT NULL
);
CREATE TABLE [Блюда]
(
[Код] INTEGER NOT NULL PRIMARY KEY,
[Название] VARCHAR(60) NOT NULL,
[Цена] MONEY NOT NULL
);
CREATE TABLE [Заказано]
(
[Код] INTEGER NOT NULL PRIMARY KEY,
[Номер заказа] INTEGER NOT NULL,
[Номер блюда] INTEGER NOT NULL
);
```
Добавление межтабличных связей:
```sql
ALTER TABLE Заказано
ADD CONSTRAINT ORDER_NO
FOREIGN KEY ([Номер заказа])
REFERENCES Заказы([Номер]);
ALTER TABLE Заказано
ADD CONSTRAINT ORDER_BL
FOREIGN KEY ([Номер блюда])
REFERENCES Блюда([Код]);
```
Запрос всех записей из связанных таблиц:
```sql
SELECT Заказано.[Номер заказа], Блюда.Название
FROM Заказано, Блюда
WHERE Заказано.[Номер блюда] = Блюда.Код;
```
или сокращенно:
```sql
SELECT [Номер заказа], Название
FROM Заказано, Блюда
WHERE Заказано.[Номер блюда] = Блюда.Код;
```
Выборка из трех таблиц:
```sql
SELECT [Номер заказа], Дата, Название
FROM Заказано, Блюда, Заказы
WHERE Заказано.[Номер блюда] = Блюда.Код
AND Заказано.[Номер заказа] = Заказы.Номер;
```
Выборка сложная - в последней строке указано по какой паре полей производится группировка:
```sql
SELECT [Номер заказа], Дата, SUM(Цена) AS Сумма
FROM Заказано, Блюда, Заказы
WHERE Заказано.[Номер блюда] = Блюда.Код
AND Заказано.[Номер заказа] = Заказы.Номер
GROUP BY [Номер заказа], Дата;
```
Запрос из запроса:
```sql
SELECT MIN(Сумма) AS МинимСумма FROM Запрос2;
```







