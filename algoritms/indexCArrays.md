# Массивы (обработка массивов)

## Массивы

Массив символов - это лишь список символов, имеющий имя. Массив чисел - это список целых чисел, имеющих имя. Вместо ссылки на каждый элемент - ссылка по общему имени массива и выделение нужного элемента в квадратных скобках. Массив может содержать элементы только одного типа.

Чтобы использовать массив, надо его объявить - присвоить ему имя, определить тип входящих в масив переменных и их количество. По этим сведениям компьютер вычислит, сколько места требуется для хранения массива, и выделит в памяти нужное число ячеек.

Объявление массива целых чисел:

    int i[25];

Объявление массива из символов сразу с его инициализацией:

    char name[5] = "Name"; //еще один элемент для нуль-символа

Можно объявлять массивы с резервом для добавления еще других элементов:

    char name[80] = "Name"; 

Объявление целочисленного массива и его инициализация меньшим кол-вом значений, чем его размер, остальные заполняются нулями:

    int val[5] = {10, 20, 30}; // {10, 20, 30, 0, 0}
    
    int val2[5] = {} // {0, 0, 0, 0, 0}

Нумерация всех элементов в массиве начинается с 0.

Объявление массива символов и инициализация его отдельными символами:

    char name[8] = {'A', 'B', 'C', 'D'};
    
Так как последний элемент не содержит нуль-символ, то этот массив содержит символы, а не строку текста.

Если же объявить массив символов этими равнозначными вариантами:

    char name[8] = {'N', 'a', 'm', 'e', '\0'}; //ручной нуль-символ
	char name[60] = "Name"; //автоматический нуль-символ

То получится строка текста. А ее можно обрабатывать как строку, выводить на печать gets() printf() с помощью %s.

При присвоении изначального значения или набора значений в момент объявления массива можно скобки где указывается размер массива оставлять пустыми - компилятор С сам посчитает:

	int ages[] = {5, 4, 23, 0};

Функция sizeof() возвращает количество байтов, которое зарезервировано на этот массив, а не количество элементов.

При объявлении и инициализации массива можно обнулить все его элементы:
```c
int ages[8] = {0};
char chName[8] = {0};
```
В массивах в элементы, которые не заполнены при инициализации обычно компилятор записывает нули.

Пример заполнения массива:
```c
int ages[5];
for (int i = 0; i < _countof(ages); i++)
{
    printf_s("Введите элемент №%d ", i);
    scanf_s(" %d", &ages[i]);
}
```

Пример вывода элементов из массива:
```c
for (int i = 0; i < _countof(ages); i++)
{
    printf_s("Элемент %d = %d\n", i + 1, *(ages + i)); //можно обойтись без квадратных скобок.
}
```
Нельзя использовать массив до инициализации его определенными значениями.

Случайный массив на С++:
```cpp
srand(time(NULL));
for (int i = 0; i < _countof(arr); i++)
    arr[i] = 10 + rand() % 80; // от 10 до 90
```

Пример заполнения элементов массива с консоли на С++:
```cpp
int arr[5] = {};
for (int i = 0; i < _countof(arr); i++)
{
    cout << "arr[" << i << "]=";
    cin >> arr[i];
 	getchar();
}
```

Пример форматированного вывода на С++:
```cpp
for (int i = 0; i < _countof(arr); i++)
    cout << setw(4) << arr[i] << ' '; //форматированный вывод на С++
```

Пример вывода элементов на С++:
```cpp
for (int i = 0; i < _countof(arr); i++)
    cout << setw(4) << arr[i] << ' '; //форматированный вывод на С++
```

Изменение массива в С++ через оператор & и цикл foreach:
```cpp
int arr[10] = {};
for (auto& el : arr)
    el = 33;
```
Символ & дает не только читать, но и изменять элемент в перечислении.




## Обработка массивов

Заполнять массивы данными можно первоначально присваиванием или в момент их объявления. Можно заполнять массивы данными, вводимиыми пользователями. Можно заполнять массивы из файлов, которые хранятся на диске.

Пример поиска элемента в массиве:
```c
int namID[10] = {313, 23, 33, 11, 21, 67, 78, 57, 65, 1};
printf_s("Введите номер для поиска:> ");
int numSearch;
int found = 0;
scanf_s(" %d", &numSearch);
for (int i = 0; i < _countof(namID); i++)
{
    if (*(namID + i) == numSearch)
    {
        found = 1;
        break;
    }
}
if (found)
{
    printf_s("Такой элемент есть в массиве!");
}
else
{
    printf_s("Такого элемента нет в массиве!");
}
```
Дополнительный пример поиска элементов в массиве:
```c
int arr[10] = {};
int i = 10;
for (auto &element : arr)
    element = i++;
int X = 11; //искомый элемент
int nX = -1; //индекс найденного элемента
for (i = 0; i<_countof(arr); i++)
    if (arr[i] == X)
    {
        nX = i;
        break;
    }
if (nX >= 0)
    cout << "Найден элемент: " << arr[nX] << " индекс: " << nX << endl;
else
    cout << "Элемент не найден!" << endl;
```

Пример поиска максимального элемента в массиве:
```c
int arr[10] = {};
int i = 10;
for (auto &element : arr)
    element = i++;
int nMax = 0;
for (i = 0; i<_countof(arr); i++)
    if (arr[i] > arr[nMax])
        nMax = i;
cout << "arr[" << nMax << "]=" << arr[nMax] << endl;
```

Пример сортировки массива методом пузырика:
```c
srand(time(NULL));
int arr[10];
for (int i = 0; i < _countof(arr); i++)
    arr[i] = rand() % 100;
printf_s("Элементы до:\n");
for (int element : arr)
{
    printf_s("%d \n", element);
}
for (int outer = 0; outer < _countof(arr) - 1; outer++)
{
    for (int inner = 0; inner < _countof(arr) - i - 1; inner++)
    {
        if (arr[inner+1] > arr[inner])
        {
            int temp = arr[inner+1];
            arr[inner+1] = arr[inner];
            arr[inner] = temp;
        }
    }
}
printf_s("Элементы после:\n");
for (int element : arr)
{
    printf_s("%d \n", element);
}
```
Пример сортировки динамического массива на С++ методом выбора:
```cpp
srand(time(NULL));
/////////////////////////////
int * arr = new int [30];
int * arr_last = &arr[29];
for (int * pI = arr; pI <= arr_last; pI++)
    *pI = rand() % 99;
/////////////////////////////
int * last = arr + SIZE_ARR - 1;
int * pI = arr;
while (pI <= last - 1)
{
    int * pMin = pI;
    int * pJ = pI + 1;
    while (pJ <= last)
    {
        if (*pJ < *pMin)
            pMin = pJ;
        pJ++;
    }
    if (pMin != pI)
    {
        int temp = *pI;
        *pI = *pMin;
        *pMin = temp;
    }
    pI++;
}
/////////////////////////////
cout << "Массив:" << endl;
for (int * pI = arr; pI <= arr_last; pI++)
    cout << *pI << ' ';
delete [] arr;
```
Пример сортировки массива методом вставки:
```cpp
void sortInsertNBinary(int arr[], int len, int * count)
{
	int key = 0;
	for (int i = 0; i < len - 1; i++)
	{
		key = i + 1;
		int temp = arr[key];
		for (int j = i+1; j > 0; j--)
		{
			if (temp < arr[j - 1])
			{
				arr[j] = arr[j - 1];
				key = j - 1;
			}
		}
		arr[key] = temp;
	}
}
//////////////////////////
	sortInsertNBinary(arr, SIZE_ARR, &count);
```

Сортировка методом слияния двух заранее отсортированных массивов основана на пошаговом сравнении двух первых еще не использованных элементов массивов, добавляя в конец третьего массива наименьший из них. После этого хвост добавляется в конец третьего массива, который уже отсортирован.

Алгоритм на языке С++:
```cpp
void merge(int arr[], int first, int mid, int last)
{
	int * tarr = new int[last - first + 1];
	int i1 = first;
	int i2 = mid + 1;
	int it = 0;
	while (i1 <= mid && i2 <= last)
		if (arr[i1] < arr[i2])
			tarr[it++] = arr[i1++];
		else
			tarr[it++] = arr[i2++];
	while (i1 <= mid)
		tarr[it++] = arr[i1++];
	while (i2 <= last)
		tarr[it++] = arr[i2++];
	for (int i = first; i<=last; i++)
		arr[i] = tarr[i-first];
    delete [] tarr;
}
void mergeSort(int arr[], int first, int last)
{
	if (first >= last) return;
	int mid = (first + last) / 2;
	mergeSort(arr, first, mid);
	mergeSort(arr, mid+1, last);
	merge(arr, first, mid, last);
}
```
Вызов сортировки методом слияния:
```cpp
	mergeSort(arr, 0, n-1);
```

Быстрая сортировка заточена под работу с большими массивами данных quicksort. Принцип работы. В данном массиве выбрать любой элемент массива X. На первом этапе расставить элементы массива так, чтоб ыслева от некоей границы находятся все числа, меньшие или равные X, а справа - большие или равные X. Далее достаточно отсорировать отдельно каждую часть массива.

Это значение X нужно выбирать чтоб оно было средним значеникм по массиву. Если нельзя выбрать - то берется случайное значение из массива.

Пример функции быстрой сортировки на С:
```c
int irand(int from, int to)
{
	return from + rand() % (to - from);
}
void quicksort( int arr[], int first, int last )
{
	int f = first, l = last;
	int middle = arr[irand(f, l)];
	while (f < l)
	{
		while ( arr[f] < middle ) f++;
		while ( arr[l] > middle ) l--;
		if ( f < l )
		{
			int t = arr[f];
			arr[f] = arr[l];
			arr[l] = t;
		}
		if ( f <= l )
		{
			f++;
			l--;
		}
	}
	if (first < l) quicksort( arr, first, l );
	if (f < last) quicksort( arr, f, last );
}
```
Использование:
```c
	srand(time(NULL));
	int arr[SIZE];
	for (int i = 0; i < SIZE; i++)
		arr[i] = irand(0, 100);
	quicksort(arr, 0, SIZE - 1);
	for (int i = 0; i < SIZE; i++)
		printf("%d ", arr[i]);
	printf("\n");
```
Пример функции быстрой сортировки с использованием указателей на С:
```c
int irand(int from, int to)
{
	return from + rand() % (to - from);
}
void qpsort( int * first, int * last )
{
	int * f = first, * l = last;
	int middle = *(f + irand(0, l - f));
	while (f < l)
	{
		while (*f < middle) f++;
		while (*l > middle) l--;
		if (f < l)
		{
			int t = *f;
			*f = *l;
			*l = t;
		}
		if (f <= l)
		{
			f++;
			l--;
		}
	}
	if (first < l) qpsort( first, l );
	if (f < last) qpsort( f, last );
}
```
Использование:
```c
    qpsort(arr, arr + SIZE - 1);
```
Сотрировка с использованием массива указателей:
```c
char books[N][20] = {"ab", "bzs", "za", "xac", "z", "a", "dss", "dsy", "dd", "rew"};
char *parr[N], *pt;
for (int i = 0; i < N; i++)
    parr[i] = &books[i];
for (int i = 0; i < N-1; i++)
    for (int j = N-2; j>=i; j--)
    {
        if ( strcmp(parr[j], parr[j+1]) > 0 )
        {
            pt = parr[j];
            parr[j] = parr[j+1];
            parr[j+1] = pt;
        }
    }
for (int i = 0; i < 10; i++)
    printf("%s ", books[i]);
puts("Отсортированное:");
for (int i = 0; i < 10; i++)
    printf("%s ", parr[i]);
```



Процесс поиска элементов в отсортированном массиве будет быстрее, так как не нужно просматривать все элементы массива, а только до определенного значения и далее прекращать поиск.

Пример двоичного поиска в отсортированном массиве:
```c
int find(int arr[], int first, int last, int key)
{
	int middle = 0;
	while (first <= last)
	{
		middle = (first + last) / 2;
		if (key < arr[middle])
			last = middle - 1;
		else if (key > arr[middle])
			first = middle + 1;
		else
			return middle;
	}
	return -1;
}
```
Использование:
```c
	int f = find(arr, 0, _countof(arr), num);
	if (f != -1)
		printf("Элемент найден: %d = %d\n", f, arr[f]);
	else
		printf("Элемент не найден!");
```

Реверс элементов массива:
```cpp
const int n = 10;
int arr[n] = {};
int i = 10;
for (auto &element : arr)
    element = i++;
int temp;
for (int i = 0; i < n/2; i++)
{
    temp = arr[i];
    arr[i] = arr[n - 1 - i];
    arr[n - 1 - i] = temp;
}
```


Случайная перестановка массива:

```cpp
srand(time(NULL));
const int n = 10;
int abas[n] = {};
int i = 1;
for (int * pI = abas; pI < abas + n; pI++) // заполнение начального массива
    *pI = i++;
int countbas = n;
int arr[n] = {}; //массив перестановленных элементов
for (int * pI = arr; pI < arr + n; pI++) //перестановка
{
    int * rnd = abas + rand() % countbas;
    *pI = *rnd;
    *rnd = *(abas + countbas - 1);
    countbas--;
}
cout << "Массив сформированный:";
for (int * pI = arr; pI < arr + n; pI++)
    cout << *pI << ' ';
```

## Двумерные массивы

Объявление массива двумерного и его заполнение пустыми значениями:
```cpp
const int N = 3, M = 4;
int arr[N][M] = {};
```
	
Заполнение массива:
```cpp
for (int i = 0; i < N; i++)
    for (int j = 0; j < M; j++)
        arr[i][j] = rand() % 61;
```

Вывод массива на экран:
```cpp
for (int i = 0; i < N; i++)
{
    for (int j = 0; j < M; j++)
        cout << setw(4) << arr[i][j] << ' ';
    cout << endl;
}
```
Перебор значений диагонали в квардатном двумерном массиве:
```cpp
puts("\nГлавная диагональ:");
for (int i = 0; i < N; i++)
    cout << setw(4) << arr[i][i] << ' ';
puts("\nПобочная диагональ:");
for (int i = 0; i < N; i++)
    cout << setw(4) << arr[i][N-i-1] << ' ';
```
Перебор всех элементов главной диагонали и под ней:
```cpp
puts("\nЭлементы под главной диагональю:");
for (int i = 0; i < N; i++)
{
    for (int j = 0; j <= i; j++)
        cout << setw(4) << arr[i][j] << ' ';
    cout << endl;
}
```
Работа указателями с массивом на С++:
```cpp
const int N = 4, M = 4;
int mtx[N][M];
int * mtx_end = mtx[N];
srand(time(NULL));
//////////////////////////////
for (int *pI = &mtx[0][0]; pI < mtx_end; pI++)
    *pI = rand() % 99;
puts("Матрица:");
for (int *pI = &mtx[0][0]; pI < mtx_end; pI++)
{
    cout << *pI << ' ';
    if ( (pI - &mtx[0][0] + 1) % N == 0 )
        cout << endl;
}
puts("\nМаксимальный элемент всей матрицы:");
int * pMax = &mtx[0][0];
for (int *pI = &mtx[0][0]; pI < mtx_end; pI++)
    if (*pI > *pMax)
        pMax = pI;
int iMax = (pMax - &mtx[0][0]) / N;
int jMax = (pMax - &mtx[0][0]) % N;
cout << "mtx[" << iMax << "," << jMax << "]=" << *pMax;
```
Работа с массивом из строк на С++:
```cpp
ifstream fin("text.txt");
if (fin)
{
	int arrstr_count = 0;
	string sbuf;
	while (getline(fin, sbuf))
		arrstr_count++;
	fin.clear();
	fin.seekg(0);
	string * arrstr = new string[arrstr_count];
	int i = 0;
	while (getline(fin, sbuf))
		arrstr[i++] = sbuf;

	for (auto i = 0; i < arrstr_count - 1; i++)
	{
		int min = i;
		for (auto j = i + 1; j < arrstr_count; j++)
			if (arrstr[j] < arrstr[min])
				min = j;
		if (min != i)
		{
			string tmps = arrstr[i];
			arrstr[i] = arrstr[min];
			arrstr[min] = tmps;
		}
	}

	cout << "Полученные и отсортированные из файла cтроки:" << endl;
	for (auto i = 0; i < arrstr_count; i++)
		cout << i << ") " << arrstr[i] << endl;
	cout << endl;
}
else
{
	cout << "Не прочитать файл";
	cin.get();
	exit(1);
}
fin.close();
```



