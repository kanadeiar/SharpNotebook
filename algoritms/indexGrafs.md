# Графы - структуры

## Получение данных из файла
Файл:
```c
0 2 4 0 0 0
2 0 9 7 0 0
4 9 0 8 1 0
0 7 8 0 3 1
0 0 1 3 0 2
0 0 0 1 2 0
```
Чтение взвешенного графа из файла:
```c
#define N 6
	int W[N][N];
	char str[64];
	FILE * fin = fopen("graf.txt", "r");
	if (fin)
	{
		int i = 0;
		while (!feof(fin))
		{
			fgets(str, _countof(str), fin);
			int j = 0;
			while (j < N)
			{
				W[i][j] = atoi(str);
				char * p = strchr(str, ' ');
				if (p!=NULL)
					strcpy(str, p+1);
				j++;
			}
			i++;
		}
		fclose(fin);
```

## Остов по графу

Нахождение остова по полученному взвешенному графу:
```c
//подготовка графа
for (int i=0; i<N; i++)
    for (int j=0; j<N; j++)
        if (W[i][j]==0)
            W[i][j]=99;
//остовное дерево
int col[N];
int ostov[N-1][2];
for (int i = 0; i<N; i++)
    col[i] = i; //раскрашивание дерева в разные цвета
int iMin, jMin;
for (int k=0; k<N-1; k++)
{
    int min = 99999;
    for (int i=0; i<N; i++)
        for (int j=0; j<N; j++)
            if (col[i]!=col[j] && W[i][j]<min) //нахождение ребра с минимальным весом
            {
                iMin = i;
                jMin = j;
                min = W[i][j];
            }
    ostov[k][0]=iMin; //добавление ребра в список выбранных
    ostov[k][1]=jMin;
    int c = col[jMin];
    for (int i=0; i<N; i++)
        if (col[i]==c)
            col[i]=col[iMin]; //перекрашивание вершин
}
//вывод остова на экран
for (int i=0; i<N-1; i++)
    printf("(%d,%d)\n", ostov[i][0]+1, ostov[i][1]+1);
```

## Кратчайший маршрут

Алгоритм Дейкстры - поиск в графе наиболее кратчайшего пути из одной точки во все остальные.

Пример:
```c
int W[N][N] = {
    {0,2,4,0,0,0},
    {2,0,9,7,0,0},
    {4,9,0,8,1,0},
    {0,7,8,0,3,1},
    {0,0,1,3,0,2},
    {0,0,0,1,2,0},
};
for (int i=0; i<N; i++)
    for (int j=0; j<N; j++)
        if (W[i][j]==0)
            W[i][j] = 999;
int from = 0; //вершина откуда считать растояния
//////////////////////////////////////////////////////
int Path[N]; //вершины откуда
int Cost[N]; //растояния стоимость
int Known[N]; //неактивные не пройденные узлы
for (int i=0; i<N; i++)
{
    Known[i]=0;
    Cost[i]=999;
    Path[i]=-1;
}
Cost[from] = 0; //вершина от которой считать расстояния
//////////////////////////////////////////////////////
for (int k = 0; k < N; k++)
{
    //нахождение минимального значения среди всех в массиве стоимостей расстояний
    int iMin = -1;
    int valMin = 999;
    for (int i=0; i<N; i++)
        if (Known[i]==0 && Cost[i] < valMin)
        {
            valMin = Cost[i];
            iMin=i;
        }
    if (iMin == -1)
        continue;
    Known[iMin]=1; //узел посещен
    //пробор по ребрам этой вершины
    for (int i=0; i<N; i++)
        if (Known[i]==0 && Cost[i] > Cost[iMin]+W[iMin][i])
        {
            Cost[i]=Cost[iMin]+W[iMin][i];
            Path[i]=iMin;
        }
}
```
Вывод всех полученных этим алгоритмом кратчайших путей до всех точек из выбранной вершины:
```c
printf("Кратчайшие пути из вершины номер %d:\n", from+1);
for (int i=0; i<N; i++)
{
    printf("До вершины %d: ", i+1);
    int v=i;
    while (Path[v]!=-1)
    {
        printf("%d ", Path[v]+1);
        v = Path[v];
    }
    printf("\n");
}
```




