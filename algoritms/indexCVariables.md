
# Переменные (типы, константы, приведение, строки, сравнение строк)

## Константы

Для определения констант используется директива #define, она аналогична литералам. Константы, определенные с помощью define, не являются переменными. Пример:
```c
#define AGELIMIT 21
#define MYNAME "Name"
``` 
Для констант в языке С нужно использовать заглавные буквы. Это позволяет быстро определять, где константы, а где - переменные.

## Переменные

Литеральные данные - это данные которые не меняются при выполнении программы. Переменные - меняются.
    
Наиболее часто используемые типы данных:
```c
char - символьные данные ('X' '#'), одиночный символ
int - целочисленные данные (-32, 5, от от -2147483648 до 2147183647)
float - числа с плавающей точкой (35.3 -34.43)
double - сверхмалое или сверхбольше значение с плавающей точкой
``` 
Дополнительные переменные на языке С++:
```c
bool - логическое значение
string - символьная строка
``` 

Имя переменной может состоять из символов в количестве от 1 до 31. Имя должно начинаться с буквы латинсокго алфавита, после него другие буквы лат.алфавита, цифры и знакип подчеркивания.

Пример использования данных в переменных:

переменная = данные;
```c
char answer = 'B';
int quant = 14, one = 4;
int rez = quant * one;
``` 

Переменным нужно давать говорящие имена, чтобы можно было сразу понять, зачем нужна та или иная переменная. Объявление переменной - это команда компилятору выделить место в памяти для хранения переменной определенного типа. Каждая переменная имеет свой тип.

## Числа
Числа без дробной части - целые числа:
```c
10 2344 -222 -434
```
>Запись числа в других системах счисления: 053-восьмеричная, 0x45-шестнадцатеричная 
    
Числа с дробным разделителем - вещественные числа с плавающей точкой
```c
10.4 2344.4343 -222.33 -0.33
```

Целые числа int. Для них обычно выделяется 4 байта памяти, то есть 32 байта. Это позволяет закодировть 2 ^ 32 различных чисел, из которых 2 ^ 31 - 1 положительных, один ноль и 2 ^ 31 отрицательных. Это значит что наибольшее положительное число 2 ^ 31 - 1 = 2147483647.

Точно определить размер переменной в байтах можно так:
```c
cout << sizeof(int);
```

Вещественные числа - это с ненулевой дробной частью нецелые числа float. Каждая переменная такого типа занимает в памяти восемь байт.

Для повышения точности нужно использовать тип double.



## Символы

Любой символ, заключенный в апостровы:
```c
'A' 'a' '4' '%' '!' '+' '=' ']'
```
Все символы заключаются в апострофы (') - одинарные кавычки. 

Группа из нескольких символов - это строка, она в двойных кавчыках:
```c
"Изучать язык весело"
```

## Булевские значения

В языке С++ добавляется еще логическая переменная bool. Она может хранить только два логичечских значения - true или false. Обявление:

bool b = false;

b = true;

В логической переменной можно хранить значение какого-либо условия и использовать его в условном операторе:
```c
int i = 10;
bool b = (i > 1) and (i < 25);
if (not b)
    cout << "Все в норме!";
else
    cout << "Все пропало!";
```




## Присваивание значений переменным

Если нужно прибавить 1 к значению переменной, то можно воспользоватся оператором составного сложения +=.

lossCount += 1;

sales *= 1.25;

amt /= 2;

days %= 3;

quality -= 5;

В языке С можно использовать операторы инкремента и декремента, которые работают только с одним аргументом.

Прибавление значению еденицы:

count++;

Вычитанию 1 из значения:

count--;

Можно написать и так:

++count++;

Если оператор находится слева, то это преинкремент или преддекремента, а после - постинктемента или постдекремента.


## Приведение типа переменной

Пример приведения типа перемнной в языке С:

(float)age;

Пример использования:

salaryBon = salary * (float)age / 150.0; //переменная age временно приводится к типу для вычисления

Пример для выражения:

value = (float)(num - 10 * yrsService);

## Размер типов данных

Для определения количества участков памяти, которые потребуются для хранения данных всех типов, используются функции sizeof(). В языке С большинство типов данных используют 4 байта для хранения целочисленных значений. Для точного определения сколько в памяти занимают целые числа и числа с правающей точкой, можно использовать функцию sizeof().

int i = sizeof(int);

int f = sizeof(float);

Пример использования:
```c
char names[] = "Name Nikolay";
int i = 7;
printf_s("Размер i %d\n", sizeof(i));
printf_s("Размер names %d\n", sizeof(names));
``` 

## Строки как массивы символов

В языке С нет строковых переменных, но есть способ сохранить строковые данные. 

В конце каждой строки в языке С добавляется специальный нуль-символ. Нексолько значений его: нуль-символ, бинарный нуль, ASII 0, \0.

Пример сохранения строки в памяти:

[С][л][о][в][о][\0] - реально строка занимает 6 байт

Длинна строки - это всегда количество символов в строке, не включая нуль-символ.

Каждый раз, когда встречается константа в двойных кавычках нужно представлять завершающий нуль-симвл на конце строки, записываемый в память компьютера.

Символьные массивы позволяют хранить в памяти компьютера строки текста. Пример:
```c
char sampl[10]; //это символьный массив
``` 
Если в него требуется сохранить слово на обычном языке, то в последний символ записывается нуль-символ. Нужно резервировать достаточное количество места для хранения самой длинной строки из тех, что потенциально потребуется плюс символ завершения строки.

Пример:
```c
char sampl[8] = "Test"; //это символьный массив
``` 
В памяти: [T][e][s][t][\0][?][?][?] начальный индекс - 0, конечный - 7

Вывод в консоль массива - через %s.

При одновременном объявлении и инициализации массива указывать длинну массива не обязательно.
```c
char sampl[8] = "Testing";
char sampl1[] = "Testing";
char sampllong[30] = "Testing"; //резервирование места для более длинного чего то
``` 

Функция strlen() возвращает длинну строки, которая находится в аргументах функции.


Функция strcat() делает конкатенацию строк. Она берет одну строку и прибавляет, то есть подставляет эту строку в конец другой строки. При этом заботится о символе конца строки \0. Нужно точно быть уверенным в том, что в первую строку поместится вторая строка.

Пример:
```c
char first[32] = "Толя";
char last[32] = " Пашин";
strcat_s(first, last);
printf_s("Имя - %s", first); //Толя Пашин
``` 

Функции puts() и gets() поедоставляют простой способо ввода и печати строк текста. Первая выводит строку текста на экран, а вторая - принимает строку текста с клаиватуры.
Функция puts() автоматически вставляет разрыв строки в конец всех распечатываемых строк. А функция gets() автоматически конвертирует нажатие клавиши Enter в нуль символ \0. Эту функцию можно использовать если нужно запросить у пользователя строки, которые содержат пробелы.

Пример использования:
```c
char fullStr[128];
char surnameName[64];
puts("Введите ваше имя фамилию:");
gets_s(surnameName);
char city[32];
puts("Город:");
gets_s(city);
puts("Ваши фамилия и имя:");
strcpy_s(fullStr, surnameName);
strcat_s(fullStr, " - годод ");
strcat_s(fullStr, city);
puts(fullStr);
``` 

Инициилизировать символьные массивы можно, но нельзя присваивать им новое знчение. Записать строку в массив символьный можно только в момент объявления массива. Если далее нужно присвоить какое либо значение новое - нужно посимвольно изменять строку, либо воспользоваться функцией strcpy().

Пример использования:
```c
char s[80];
char s1[] = "Привет!";
char s2[] = "Вася";
strcpy_s(s, _countof(s), s1); //Привет!
``` 
Так как функция принимает в качестве аргументов адреса в памяти, то можно производить вставку строки в строку:
```c
char s[80] = "Привет!";
strcpy_s(&s[6], _countof(s), "Вася"); //ПриветВася
``` 
Или так:
```c
char s[80] = "Синий автобус";
char s1[] = "Трезвый Вася";
strcpy(s + 6, s1 + 8);
puts(s); //Cиний вася
``` 
Можно использовать для "удаления" символов так:
```c
char s[80] = "Вася не врет!";
strcpy(s + 4, s + 7);
puts(s); //Вася врет!
``` 
Дополнительная функция strncpy() предназанчена тоже для копирования символов из массива в массив, но она копирует не всю строку, а указанное количество символов и в конец не добавляет символ конца строки.

Пример использования strncpy() для изменения строки:
```c
char s[] = "Кот Мурзик любит молоко!";
char s1[] = "Пес Барсик любит мясо";
strncpy( s+4, s1+4, 6);
puts(s); //Кот Барсик любит молоко!
``` 
Можно легко получить подстроку как часть строки:
```c
char s[] = "Кот Мурзик любит молоко!";
char s2[20];
strncpy( s2, s+4, 6);
s2[6] = '\0';
puts(s2); //Мурзик
``` 
Вставка фрагмента в строку возможна только используя буфер:
```c
char s[80] = "Кот мяукает!";
char s1[30];
strcpy( s1, s+4);
strcpy(s+4, "громко ");
strcat(s, s1);
puts(s); //Кот громко мяукает!
``` 
С помощью функции strchr() можно найти один символ в строке. Фуккция возвращает указатель на символ в строке. Если символов несколько, то функция найдет первое вхождение символа в строку. Чтобы искать с конца строки нужно применять strrchr.

Пример:
```c
char str[] = "Пример текста";
char* p = strchr(str, 'и');
if (p == NULL)
    printf("Символ не найден.\n");
else
    printf("Номер первого символа 'и': %d\n", p - str); //2
```
Пример реверсивного поиска:
```c
char* p = strrchr(str, 'т');
```

С помощью функции strstr можно получить позицию подстроки в строке.

Пример:
```c
char str[] = "Пример текста.";
char * p;
p = strstr(str, "текст");
if (p == NULL)
    printf("Слово не найдено.");
else
    printf("Это слово начинается с s[%d]\n", p - str);
``` 

Преобразования символьные массивы <-> числа.

Для этих целей в С есть функции atoi() & atof() - преобразования символьных строк в целые числа и вещественные числа. 

Примеры:
```c
char s[] = "123";
int n = atoi(s);
printf("%d\n", n); // 123
char s2[] = "123.456";
float nf = atof(s2);
printf("%.2f\n", nf); // 123.46
``` 
Обратное преобразовение делается с использованием функции sprintf(), которая выводит результат преобразования в строку:
```c
char str[90], str2[90];
int num = 123;
float fnum = 123.375f;
sprintf_s(str, "%d", num);
sprintf_s(str2, "%10.2f", fnum);
printf("n1:%s  n2:%s", str, str2); //n1:123  n2:    123.38
```

## Строки как объекты

В язык С++ доабавляется еще один тип переменных строки string. По сути являются объектами. Символьная строка - последовательность символов. Они позволяют хранить массив символов.

Объявление строки:

std::string name;

Объявление с инициализацией:

string nam("Andrey");

string s = "Andrey";

string s4(5, 'j'); //пять одинаковых сиволов

ВНИМАНИЕ! КОПИЯ другой строки, НЕ УКАЗАТЕЛЯ КОПИЯ:
```c
string def = "default";
string cloned = def;
def += " none";
cout << def << endl; //default none
cout << cloned << endl; //default
```
Для работы со строкой в консольном вводе/выводе нужно ползоваться функцией getline():
```c
cout << "Введите свое имя:> ";
string name;
getline(cin, name);
cout << "Привет, " << name + " Грозный" << '!' << endl;
```
Строки можно складывать друг с другом. И получать длинну строки:
```c
name += " Грозный";
int len = name.length();
```
Возможно измененеие отдельного символа в строке:
```c
name[1] = 's';
```
Получение подстроки из строки:
```c
string s = "Строка";
string s1 = s.substr( 2, 2);
cout << s1 << endl; // ро
```
Удаление символов из строки
```c
string s = "Строка";
s.erase(2,2);
cout << s << endl; //Стка
```
Удаление всех символов и освобождение занимаемфх ими пямяти. После этого size = 0, а метод empty - истина:
```c
string s = "информатика";
s.clear();
cout << s.empty() << endl; // 1
```
Вставка новой строки в уже имеющуюся:
```c
string s = "информатика";
s.insert( 3, "номикрон");
cout << s << endl; //инфономикронорматика
```
Поиск в символьных строках - find(). Возвращает индекс первого найденного символа, а при поиске подстроки - индекс первого символа первого вхождения подстроки. Если символ не найдет, то возвращается string::npos что (int) дает -1:
```c
string s = "Пример строки";
int n = s.find('м');
if (n != string::npos)
    cout << "буква м в позиции номер " << n << endl; // 3
else
    cout << "буквы м нет" << endl;
```
слово начиная с определенной позиции:
```c
string s = "Пример строки";
int n = s.find("строк");
if (n >= 0)
    cout << "слово \"строк\" м в позиции номер " << n << endl; // 7
else
    cout << "слова \"строк\" нет" << endl;
```
Поиск с конца строки:
```c
string s = "Пример строки";
int n = s.rfind('к');
if (n >= 0)
    cout << "буква к в позиции номер " << n << endl; // 11
else
    cout << "буквы к нет" << endl;
```
Замена в символьных строках часть подстроки на заданную - replace(). Первый аргумент - индекс первого заменяемого символа, второй - количество заменяемых символов, третий - строка, которую нужно поставить на их место.
```c
string s = "Пример/строки";
s.replace(s.find('/'), 1, " ");
cout << "Строка: " << s << endl;
```

Преобразования строки <-> числа. 

Функции stoi() и stof(). Функци останавливают преобразование на том символе, которые не соответствует типу данных.

Пример:
```c
string s = "12.12ggg";
int num = stoi( s ); //12
float numFl = stof( s ); //12.12
cout << "Числа: " << num << " & " << numFl << endl;
```
Преобразование из числа в строку - функция to_string():
```c
int n = 12;
string sN = to_string( n ); //12
float nF = 12.12;
string sF = to_string( nF ); //12.120000
cout << "Строки: " << sN << ' ' << sF << endl;
```
Преобразование происходит по умолчанию, если нужно форматированное преобразование - подключать заголовочный файл sstream "".

Пример использования:
```cpp
ostringstream sbuf; //вспомогательный выходной буферный поток символов
int num = 123;
sbuf << num;
string sNum = sbuf.str(); // 123
sbuf.str(""); //очистка потока
float numF = 123.123456;
sbuf.width(10);
sbuf << fixed << setprecision(2) << setw(8) << numF;
string sFlt = sbuf.str(); //"   123.123"
sbuf.str(""); //очистка потока
double numD = 0.0000232342;
sbuf << scientific << setprecision(6) << setw(16) << numD;
string sScientific = sbuf.str(); //2.323420e-05 - научный формат
cout << "Числа: " << sNum << ' ' << sFlt << ' ' << sScientific << endl;
```

## Функции работы с символьными массивами

Функия isapha() возвращает значение ИСТИНА (1), если символ в скобках - символ алфавита от a до z (или от A до Z) и ЛОЖЬ (0) если значение в скобках другой символ.

Пример применения:
```c
char ch = 'a';
if (isalpha(ch))
{
    printf_s("Введена буква\n");
}
``` 
Функция isdigit() возвращает ИСТИНА (1) если символ в скобках - число от 0 до 9.

Пример применения:
```c
char ch = '1';
if (isalpha(ch))
{
    printf_s("Введена цифра\n");
}
``` 
Функции isupper() и islower() информируют о том, содержит ли переменная символ верхнего или нижнего регистра.

Пример примененеия:
```c
char ch = 'A';
if (isupper(ch))
{
    printf_s("Буква верхнего регистра\n");
}
``` 

Функции toupper() и tolower() возвращают переданный им аргумент с исзмененным регистром. Первая функция возвращает аргумент, наход в скобках в виде символа верхнего регистра, а вторая - нижнего.

Пример:
```c
char answer = 'y';
if (toupper(answer) == 'Y')
{
    printf_s("да\n");
}
else
{
    printf_s("нет\n");
}
``` 

Алгоритм рекурсивного перебора различных комбинаций слов по алфавиту. Когда нужно перебрать все слова, и определять все символы последовательно во всех словах.

Пример - алфавит из четырех букв, слово длинной три символа, всего - 64 различных комбинаций. Число комбинаций равно W ^ L, где W - количество букв в алфавите, L - длинна слова.
```c
void ReGetWord(const char abc[], char word[], int n)
{
	int i;
	if (n >= strlen(word))
	{
		puts(word);
		return;
	}
	for (int i = 0; i < strlen(abc); i++)
	{
		word[n] = abc[i];
		ReGetWord(abc, word, n + 1);
	}
}
////////////////////////////////////////////////
	char word[] = "...";
	ReGetWord("АБВГ", word, 0);
``` 
Тоже на С++:
```cpp
void ReGetWord(string abc, string &word, int n)
{
	int i;
	if (n >= word.size())
	{
		cout << word << endl;
		return;
	}
	for (int i = 0; i < abc.size(); i++)
	{
		word[n] = abc[i];
		ReGetWord(abc, word, n + 1);
	}
}
////////////////////////////////////////////////
	string word = "...";
	ReGetWord("АБВГ", word, 0);
``` 

## Сравнение строк

При сравнении строк используются коды символов.

"ПАР" < "Пар" < "пар" < "парк" 

"5TANK" < "TANK" < "Tank" < "tank" < "ПАР" < "Пар" < "пар"

Для сравнения строк в языке С используется функция strcmp(). Функция возвращает разность кодов первых различных символов двух строк или 0, если все символы совпали. Пример:
```c
const int n = 4;
char st[80];
char s[n][80] = {"абв", "ААА", "Ббб", "Ааа"};
for (int i=0; i<n-1; i++)
    for (int j=n-2; j>=i; j--)
        if (strcmp(s[j], s[j+1]) > 0)
        {
            strcpy_s(st, s[j]);
            strcpy_s(s[j], s[j+1]);
            strcpy_s(s[j+1], st);
        }
for (int i=0; i<n; i++)
    puts(s[i]);
``` 
Пример на С++:
```cpp
const int n = 4;
string st;
string s[n] = {"абв", "ААА", "Ббб", "Ааа"};
for (int i=0; i<n-1; i++)
    for (int j=n-2; j>=i; j--)
        if (s[j] > s[j+1])
        {
            st = s[j];
            s[j] = s[j+1];
            s[j+1] = st;
        }
for (int i=0; i<n; i++)
    cout << s[i] << endl;
``` 