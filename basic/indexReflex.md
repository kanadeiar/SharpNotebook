# Рефлексия

В процессе работы приложения может возникнуть потребность в выявлении типов во время выполнения приложения. Каждая сборка .NET содержит описание набора используемых классов, интерфесов, методов, свойств, и пр. 

Рефлексия "самопознание" позволяет определить составные элементы сборки. Это процесс обнаружения типов во время работы программы, используя дружественную объектную модель. Можно также получать набор интерфейсов этого типа, параметры метода и другие детали.

Основные члены пространства рефлексии Object.Reflexion:
```csharp
Assembly        Класс содержит несколько членов, которые позволяют загружать, исследовать и манипулировать сборкой
AssembleName    Идентичность сборки
EventInfo       Информация о заданном событии
FieldInfo       Информация о заданном поле
MemberInfo      Определяет общее поведение типов для остальных членов
MethodInfo      Информация о заданном методе
Module          Доступ к заданному модулю внутри многофайловой сборки
ParameterInfo   Информация о заданном параметре
PropertyInfo    Информация о заданном свойстве
```

Исследование метаданных типа с помощью System.Type:
```csharp
IsAbstract, IsArray, IsClass, IsCOMObject, IsEnum, IsGenericTypeDefifnition, IdGenericParameter, IsInterface, IsPrimitive, IsNestedPrivate, IsNestedPublic, IsSealed, IsValueType - выяснение базовых характеристик типа
GetConstructor, GetEvents, GetFields, GetInterfaces, GetMembers, GetMethods, GetNestedTypes, GetProperities - получение массива интересующих элементов.
FindMembers - массив объектов MemberInfo на основе указанного критерия поиска
GetType - статический метод возвращает экземпляр Type с заданным строковым имененм
InvokeMember - выполнение "поздего связывания" для указанного элемента
```















## Получение типа
```csharp
Type type = typeof(MyClass);
```
В роли аргумента может выступать имя класса - как встроенного, так и созданного пользователем.
Вертаемый тип - Type.

```csharp
Type type = variable.GetType();
```
В роли аргумента здесь выступает экземпляр класса, результат будет такой-же.
Вертаемый тип - Type.

Пример использования:
```csharp
Type type = typeof(int);
Console.WriteLine(type);
int i = 0;
Type type2 = i.GetType();
Console.WriteLine(type2);
Console.WriteLine(i.GetType().Attributes);
var types = typeof(int).GetFields();
foreach (var v in types)
{
    Console.WriteLine(v);
}
```



