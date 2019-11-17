# Библиотека классов

## Класс математических операций
```csharp
Math.Pow(a,b); //возаращает а в степери b
```

## Класс работы с массивом 
Array

Имя метода      Тип             Описание


Clear           Статический     Присваивает элементам массива значение 0,

                                false или null в зависимости от типа элементов

GetLength       Нестатический   Получает целое число, представляющее 

                                количество элементов в заданном измерении

Sort            Статический,    Сортирует элементы во всем одномерном массиве Array

                перегруженный

                                
                                


## Класс случайных значений
```csharp
Random rnd = new Random();
int variable = rnd.Next(0, 100);
```

## Получение прошедшего времени
```csharp
DateTime start=DateTime.Now;
System.Threading.Thread.Sleep(20);// делаем паузу
DateTime finish=DateTime.Now;
Console.WriteLine(finish-start);
```




## Замер скорости выполения кода:
```csharp
Stopwatch stopwatch = new Stopwatch();
stopwatch.Start();
/////////код для замера
stopwatch.Stop();
var elapsedTime = stopwatch.Elapsed; //результат работы
```

