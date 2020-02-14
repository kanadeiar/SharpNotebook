# Структуры языка C++

## тип С++ vector

На языке С++ можно использовать класс vector - динамический массив с возможностью расширения из библиотеки шаблонов STL.

Объявление:
```cpp
vector<int> arr;
```

Пример работы с массивом в С++:
```cpp
#include <vector>
///////////////////////////
	vector<int> arr;
	cout << arr.size();
	for (int i = 0; i<10; i++)
		arr.push_back(i+1);
	for (int i = 0; i<10; i++)
		cout << arr[i] << " ";
```
Пример работы с двумерным массивом на С++:
```cpp
#define N 10
#define M 10
typedef vector<int> vint;
vector<vint> varr;
..........................................
	varr.resize( N ); //количество
	for (int i = 0; i < N; i++)
		varr[i].resize( M ); //количество подмассивов
	for (int i = 0; i < M; i++)
		for (int j = 0; j < N; j++)
			varr[i][j] = (i+1)*(j+1);
	for (int i = 0; i < N; i++)
	{
		for (int j = 0; j < M; j++)
		{
			printf("%3d ", varr[i][j]);
		}
		printf("\n");
	}
	varr.clear();
```


## тип С++ map

Подходит организации словаря. Для работы с ним нужно включить в файл заголовочный файл "<map>".
    
Объявление:
```cpp
#include <map>
map <string, int> marr;
```
Пример работы:
```cpp
srand(time(NULL));
map<int, int> MapArr;
int tmp;
for (int i = 0; i < 100; i++)
{
    tmp = 1 + rand() % 30;
    int p = MapArr.count(tmp); //поиск элемента в словаре, 1 - элемент ест, 0 - нет
    if (p == 1)
        MapArr[tmp]++; //увеличение подсчета элементов
    else
        MapArr.insert(pair<int,int> (tmp,1)); //добавление нового элемента, это можно не использовать т.к. если элемента нет, то MapArr[i]++; создаст новый элемент
}
cout << "Результат:" << endl;
map <int, int>::iterator it; //вывод результатата через итератор
for (it = MapArr.begin(); it != MapArr.end(); it++)
    cout << it->first << " = " << it->second << endl;
```


