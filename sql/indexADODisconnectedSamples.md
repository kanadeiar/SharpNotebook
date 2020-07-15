# ADO.NET Автономный уровень Примеры

Код SQL генерации тестовой базы данных:
```csharp
CREATE TABLE [dbo].[Person] (
    [Id]   INT           IDENTITY (1, 1) NOT NULL,
    [Fam]  NVARCHAR (64) NOT NULL,
    [Name] NVARCHAR (64) NOT NULL,
    [Age]  INT           NOT NULL,
    CONSTRAINT [PK_Id_Person] PRIMARY KEY CLUSTERED ([Id] ASC)
);
CREATE TABLE [dbo].[Child] (
    [Id]       INT          IDENTITY (1, 1) NOT NULL,
    [Fam]      NVARCHAR (64) NOT NULL,
    [Name]     NVARCHAR (64) NOT NULL,
	[Age] INT NOT NULL,
    [PersonId] INT          NOT NULL,
    CONSTRAINT [PK_Id_Child] PRIMARY KEY CLUSTERED ([Id] ASC),
    CONSTRAINT [FK_Child_To_Person] FOREIGN KEY ([PersonId]) REFERENCES [dbo].[Person] ([Id])
);

INSERT INTO Person([Fam], [Name], [Age])
VALUES (N'Иванов', N'Иван', 20),
(N'Петров', N'Петр', 22),
(N'Сидоров', N'Сидор', 18);
INSERT INTO Child([Fam], [Name], [Age], [PersonId])
VALUES (N'Михаилов', N'Михаил', 3, 1),
(N'Навозов', N'Дмитрий', 13, 1),
(N'Логинов', N'Логин', 3, 2);
```





