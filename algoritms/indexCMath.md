# Арифметика (операции и функции)

## Математические операции

Обычное математическое выражение:

newVal = oldVal - factor;

newVal = maxim / 1.3 * 100.0;

Значение присваивается левой переменной.

Выражение можно поместить в функцию printf().
```c
printf_s("Пример вычисления %d этот", age + 3);
``` 
Отсечение дробной части:

int vari = 10 / 3;

Остаток от деления (операция деление по модулю):

int var2 = 10 % 3;

Порядок выполнения операций в языке C:
```c
Уровень - Оператор - Ассоциативность
1 - ()скобки, []элемент из массива и .ссылка на член структуры - слева направо
2 - -знак отриц.числа, ++инкремент, --декремент, &взятие адреса, *разыменование указателя, sizeof(), !логическое НЕ - справа навлево 
3 - *умножение, /деление, %деление по модулю - слева направо
4 - +сложение, -вычитание - слева направо
5 - <меньше, <=меньше или равно, >больше, >=больше или равно - слева направо
6 - ==равно, !=меньше - слева направо
7 - && логическое И - слева направо
8 - || логическое ИЛИ - слева направо
9 - ?: оператор условия - справа налево
10 - =, *=, /=, %=, |=, -= операторы присваивания - справа налево
11 - , оператор-запятая - слева направо
```
С помощью скобок можно установить нужный порядок выполения оперторов.

Результатом всех выражений С являются значения, а это значит что результатом присваивания одной переменной другой будет значение другой переменной. Можно делать так:

a = b = c = 99; и так: a = 2 * (b * 2);

Если в выражения входят переменные разных типов, то происходит автоматическое преобразование (приведение) типа. Если нужно чтоб при делении целых чисел получить вещественный результат, то нужно чтоб хотяб одно из двух чисел было вещественными.

Операции на языке С++:
```c
float x;
x = 3./4;
x = 3/4.;
cout << x << endl;
```
Для преобразования целого значения в вещественное используется функция float:
```c
float x;
int a = 3, b = 5;
x = float(a) / float(b);
cout << x << endl;
```



## Математические функции

При работе с вещественными числами часто приходится округлять их до ближайших целвх чисел. Для этого можно использовать одну из двух встроенных функций:

int(x) - отбрасывание дробной части x.

round(x) - округление вещественного числа х к ближайшему целому числу.

Функции floor() и ceil() можно назвать функцие пола и потолка, они снижают или возносят нецелые числа к ближайшему целому значению.

Пример:
```c
float val1 = floor(18.5); //18.0
float val2 = floor(-18.5); //-19.0
float val3 = ceil(18.5); //19.0
float val4 = ceil(-18.5); //-18.0
```

Функция fabs() возвращает абсолютное значение числа с плавающей точкой. 

Пример:
```c
printf_s("Абсолютное знач. числа 23.4 - %.1f.\n", fabs(23.4));
printf_s("Абсолютное знач. числа -23.4 - %.1f.\n", fabs(-23.4));
```

Функция возведения в степень pow() и функция квадратного корня из числа sqrt().

Пример:
```c
printf_s("10 в 3 степени: %.1f\n", pow(10.0, 3.0));
printf_s("Квадратный корень из 64: %.1f\n", sqrt(64));
```

Тригонометрические функции языка С:
```c
Функция             Описание
cos(x)              Косинус угла х
sin(x)              Синус угла х
tan(x)              Тангенс угла х
acos(x)             Арккосинус угла х
asin(x)             Арксинус угла х
atan(x)             Арктангенс угла х
```
Конвертация из градусов в радианы:

radians = degrees * (3.14159 / 180.0);

Логарифмические функции языка С:
```c
Функция             Описание
exp(x)              Значение экспоненты, основания натуркльного логарифма, возведенного в степень, заданную выражением x (e ^ x).
log(x)              Натуральный логарифм аргумента х. Записывается как ln(x).
log10(x)            Логарифм по основанию 10 аргумента х. Записыватеся как lg(x).
```

Примеры использования функций:
```c
printf_s("Квадратный корень из 36: %.1f\n", sqrt(36.0));
printf_s("4 в 3-ей степени: %.1f\n", pow(4.0, 3.0));
printf_s("Косинус угла 60 градусов: %.3f\n", cos((60*(3.14159/180))));
printf_s("е в степени 2: %.3f\n", exp(2));
printf_s("Натуральный логарифм 5: %.3f\n", log(5));
```

Функция rand() позволяет генерировать в языке С случайные числа. Она возвращает случайное число в диапазоне от 0 до 32767. Для уменьшения дипазона случайных чисел можно воспользоватся оператором деления по модулю %. Например от 1 до 3: (rand() % 2) + 1.

Сначало нужно генератору случайных чисел установить базу, исходя из которой функция rand() будет отсчитывать случайные числа. В аргумент нужно передать точное значение времени дня, тогда это будет случайное значение.

Пример использования случайных чисел:
```c
srand(time(NULL));
int dice1 = (rand() % 5) + 1; // от 1 до 6
printf_s("Значение игральной кости: %d", dice1);
```

Получение вещественного числа на отрезке от a до b:
```c
float a = 1.221;
float b = 99.1;
float x = a + (b - a) * rand() / float(RAND_MAX);
cout << x << endl;
```
