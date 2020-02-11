# Файлы (последовательная и произвольная работа с файлами)

## Текстовые последовательные файлы

В файлы этого типа можно считывать и записывать данные в порядке следования данных.

Открытие файла производится функцией fopen(). 

Указатель на файл объявляется следующим образом:

FILE * f_pointer;

После объявления указателя, его можно ассоциировать с нужным файлом с помощью fopen().

Пример открытия файла:

В файлы этого типа данные можно записывать и считывать в разные участки этого файла. Пример работы с файлом:
```c
int main(int argc, const char* argv[])
{
	SetConsoleCP(1251);
	SetConsoleOutputCP(1251);
	FILE * fptr;
	fopen_s(&fptr, "c:\\Temp\\test.txt", "w");
	if (fptr)
	{
		printf_s("Файл создан!");
	}
	else
	{
		printf("Не удалось создать файл!");
		exit(1);
	}
	fclose(fptr);
```
Если по каким то причинам не удалось открыть файл, то вызов функции возвращает 0.

Строковые модификаторы функции fopen():
```c
"w" - режим записи, создание нового файла, вне зависимости от того, существует он или нет.
"r" - режим чтения, чтение существующего файла, если файл не существует, будет ошибка
"a" - режим добавления (дозаписи) существующего файла или создания нового, если файл не существует
```

Для дозаписи в файл следует использовать функцию fprintf(). Первым в ней аргументом - указатель на файл.

Пример записи в файл некоторых данных:
```c
#define SIZE_BOOKS 3
struct Book
{
	char name[65];
	float price;
	int pages;
};
FILE *fptr;
int main(int argc, const char* argv[])
{
	SetConsoleCP(1251);
	SetConsoleOutputCP(1251);
	Book books[SIZE_BOOKS];
	for (int i = 0; i < SIZE_BOOKS; i++)
	{
		strcpy_s(books[i].name, _countof(books[i].name), "Test");
		books[i].price = 1.1 * i;
		(books + i)->pages = 100 * i;
	}	
	fopen_s(&fptr, "c:\\Temp\\books.txt", "w");
	if (fptr)
	{
		printf_s("Файл создан!");
	}
	else
	{
		printf("Не удалось создать файл!");
		exit(1);
	}
	fputs("Коллекция книг:\n", fptr);
	for (int i = 0; i < SIZE_BOOKS; i++)
	{
		fprintf_s(fptr, "%d: %s\n", (i + 1), (books + i)->name);
		fprintf_s(fptr, "Содержит %d страниц и стоит %.2f\n", books[i].pages, books[i].price);
	}
	fclose(fptr); //всегда закрывать открытые файлы
```

Для чтения информации из файла следует использовать функцию fgets(). Она требует указания длины массива, в который производится считывание данных. При работе с функцией нужно быть бдительным и проверять положение указателя, что он не в конце файла.

Функция feof() - возвращает истинное значсение, если файловый курсор указывает на конец файла.

Пример чтения строк из записанного ранее файла:
```c
FILE * fptr;
int main(int argc, const char* argv[])
{
	SetConsoleCP(1251);
	SetConsoleOutputCP(1251);
    char line[254]; //хранение вводимых строк
    fopen_s(&fptr, "c:\\Temp\\books.txt", "r");
    if (fptr)
    {
        while (!feof(fptr))
        {
            fgets(line, _countof(line), fptr);
            if (!feof(fptr))
            {
                printf_s(line);
            }
        }
        fclose(fptr);
    }
    else
    {
        puts("Не удалось открыть файл для чтения!");
        exit(1);
    }
```
Другой вариант чтение чисел РЕШНИЕ ОШИБКИ с учетом ошибки если в конце файла стоит символ перевода строки, то последнее число будет учтено дважды:
```c
FILE * fptr;
char line[254]; //хранение вводимых строк
fopen_s(&fptr, "c:\\Temp\\letters.txt", "r");
if (fptr)
{
    int n;
    int x;
    while (1)
    {
        n = fscanf_s(fptr, "%d", &x);
        if (n < 1)
            break;
        printf("%d\n", x);
    }
    fclose(fptr);
}
else
{
    puts("Не удалось открыть файл для чтения!");
    exit(1);
}
```

Пример дозаписи строки в файл:

```c
FILE * fptr;
int main(int argc, const char* argv[])
{
	SetConsoleCP(1251);
	SetConsoleOutputCP(1251);
	fopen_s(&fptr, "c:\\Temp\\books.txt", "a");
	if (fptr)
	{
		printf_s("Файл создан!");
	}
	else
	{
		printf("Не удалось создать файл!");
		exit(1);
	}
	fputs("Нужно больше книг!:\n", fptr);
	fclose(fptr); //всегда закрывать открытые файлы
```
Нельзя производить чтение файла без использования функции feof(), так как возможно, уже прочитана последняя строка файла.

## Текстовые файлы произвольного доступа

Физическое размещение файла не определяет его тип. Можно создать файл последовательно, а затем читать и изменять его произвольно.

Чтобы читать и записывать в файл произвольно, нужно открыть его в режиме произвольного доступа.

Строковые модификаторы функции fopen():
```c
"w+" - открывает новый файл для чтения и записи
"r+" - открывает существующий файл для чтения и записи
"a+" - открывает файл в дозаписи, но позволяет вернуться назад по тексту и совершить чтение или запись по мере необходимости
```

Для переещения по файлу следует использовать функцию fseek(). Функция эта смещает указатель по фалу таким образом, что для записи и чтения становятся доступны те участки файла, которые были недоступными при последовательном доступе.

Формат функции:

fseek(pFile, longVal, начало)

указатель на файл (в произвольном режиме), количество байтов, которые нужно пропустить (вперед или назад) при перемещении по файлу, последний аргумент - это одно из значений, сообщающее откуда нужно начинать поиск нужно фрагмента функцией fseek().
```c
начало          описание
SEEK_SET        начало файла
SEEK_CUR        текущая позиция в файле
SEEK_END        конец файла
```
Если установить указатель на конец файла SEEK_END и начать записывать, то новые данные будут добавлены в конец файла. Если иначе - то новые данные затрут уже имеющиеся.

Пример работы с файлом в режиме произвольного доступа - запись и чтение:
```c
FILE * fptr;
int main(int argc, const char* argv[])
{
	SetConsoleCP(1251);
	SetConsoleOutputCP(1251);
	fopen_s(&fptr, "c:\\Temp\\letters.txt", "w+");
	if (fptr)
	{
		printf_s("Файл создан!");
	}
	else
	{
		printf("Не удалось создать файл!");
		exit(1);
	}
	for (char lett = 'A'; lett <= 'Z'; lett++)
	{
		fputc(lett, fptr);
	}
	puts("Данные в файл записаны");
	puts("Данные из файла в обратной последовательности:");
	fseek(fptr, -1, SEEK_END);
	for (int i = 26; i > 0; i--)
	{
		printf_s("Буква - %c\n", fgetc(fptr));
		fseek(fptr, -2, SEEK_CUR);
	}
	fclose(fptr);
```
Пример работы с файлом в режиме произвольного доступа - изменение файла:
```c
FILE * fptr;
int main(int argc, const char* argv[])
{
	SetConsoleCP(1251);
	SetConsoleOutputCP(1251);
	fopen_s(&fptr, "c:\\Temp\\letters.txt", "r+");
	if (fptr)
	{
		printf_s("Файл открыт!");
	}
	else
	{
		printf("Не удалось создать файл!");
		exit(1);
	}
	printf_s("Какую букву заменить? (1-26)?:> ");
	int pos;
	scanf_s(" %d", &pos);
	fseek(fptr, pos - 1, SEEK_SET);
	fputc('*', fptr); //замена символа на другой
	fseek(fptr, 0, SEEK_SET);
	puts("Измененный файл:");
	for (int i = 0; i < 26; i++)
	{
		printf_s("Буква - %c\n", fgetc(fptr));
	}
	fclose(fptr);
```

## Текстовые файлы на языке С++

На языке С++ работа с текстовыми файлами используются файловые потоки ввода-вывода, которые подключеются с помощью заголовочного файла fstream.

Входной поток - чтение - ifstream, выходной поток - запись - ofstream. Для открытия файла метод open(), для закрытия - close(). В случае ошибки значение будет NULL. Функция eof() используется для определения конца файла.

Пример:
```cpp
int a = 12345;
int a1 = 54321;
ofstream fout;
fout.open("c:\\Temp\\letters.txt");
if (fout)
{
    fout << a << endl << a1;
}
else
    printf("Не удалось записать файл!");
fout.close();
ifstream fin;
fin.open("c:\\Temp\\letters.txt");
if (fin)
{
    int b;
    while (!fin.eof())
    {
        fin >> b;
        cout << b << endl;
    }
}
else
    printf("Не удалось записать файл!");
fin.close();
```
Пример записи чисел:
```cpp
ofstream fout;
fout.open("test.txt");
for (int i = 0; i<n; i++)
    fout << arr[i] << endl;
fout.close();
```

Пример чтения строк:
```cpp
string s;
ifstream fin;
fin.open("c:\\Temp\\books.txt");
while (getline(fin, s))
{
    cout << s << endl;
}
fin.close();
```

## Бинарные файлы

Структура на С:
```c
typedef struct 
{
	char author[40];
	char title[80];
	int count;
} Book;
```
Запись в файл одной записи и массива:
```c
FILE * fout;
Book book;
Book books[10];
strcpy(book.author, "Пупкин А.И.");
strcpy(book.title, "Стихи");
book.count = 10;
for (int i = 0; i < 10; i++)
    books[i] = book;
fout = fopen("books.dat", "wb");
fwrite(&book, sizeof(Book), 1, fout);
fwrite(&books[0], sizeof(Book), 10, fout);
fclose(fout);
```
Чтение одной записи и массива:
```c
FILE * fin;
Book book2;
Book books2[10];
fin = fopen("books.dat", "rb");
fread(&book2, sizeof(Book), 1, fin);
printf("Книга 1-я: %s %s %d\n", book2.author, book2.title, book2.count);
int cou = fread(&books2, sizeof(Book), 10, fin);
printf("Прочитано %d книг:\n", cou);
for (int i = 0; i < 10; i++)
    printf("%s %s %d\n", books2[i].author, books2[i].title, books2[i].count);
fclose(fin);
```
Запись в файл на С++:
```cpp
Book B;
B.author = "Пупкин";
B.title = "Стихи";
B.count = 10;
Book books[10];
for (int i = 0; i < 10; i++)
    books[i] = B;
ofstream fout;
fout.open("books.dat", ios::binary);
fout.write( (char*)&B, sizeof(Book) );
fout.write( (char*)books, sizeof(Book)*10 );
fout.close();
```
Чтение из файла на С++:
```cpp
Book B2;
Book books2[10];
ifstream fin;
fin.open("books.dat", ios::binary);
fin.read( (char*)&B2, sizeof(Book) );
fin.read( (char*)books2, sizeof(Book)*10 );
fin.close();
cout << "Книга: " << B2.author << " " << B2.title << " " << B2.count << endl;
int M = fin.gcount() / sizeof(Book);
cout << "Прочитано " << M << " книг:" << endl;
for (auto it : books2)
    cout << it.author << " " << it.title << " " << it.count << endl;
```




