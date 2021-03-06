# Алгоритмы простые

## Простые числа

Определение что число - простое.
```c
bool isSimple( int n )
{
	int k = 2;
	while ( k*k <= n && n%k != 0 )
		k++;
	return ( k*k > n );
}
```
Решето Эратосфена.
```c
int arr[N + 1];
int i, k;
for (i = 2; i<=N; i++)
    arr[i] = 1;
k = 2;
while (k * k <= N)
{
    if (arr[k])
    {
        i = k*k;
        while (i <= N)
        {
            arr[i] = 0;
            i += k;
        }
    }
    k++;
}
for (i = 2; i<=N; i++)
    if (arr[i])
    {
        printf("%d ", i);
    }
```

## НОД

Алгоритм Евклида.

Нахождение наибольшего общего делителя двух натуральных чисел. Заменяь большее из двух заданных чисел на их разность до тех пор, пока они не станут равными. Полученное значение - НОД:

```c
int algEvkl(int a, int b)
{
	while (a != b)
	{
		if (a > b)
			a -= b;
		else
			b -= a;
	}
	return a;
}
```
Модифицированный алгоритм Евклида:

Заменять большее из двух заданных чисел на остаток от деления большего на меньшее, пока остаток не станет равным нулю:

```c
int algEvkl(int a, int b)
{
	while (a != 0 && b != 0)
	{
		if (a > b)
			a %= b;
		else
			b %= a;
	}
	return a + b;
}
```

Алгоритм НОД с использованием рекурсии:
```c
int NOD (int a, int b)
{
	if ( a == 0 || b == 0 )
		return a + b;
	if ( a > b )
		return NOD( a % b, b );
	return NOD( a, b % a );
}
```

## НОК

Наибольшее общее кратное двух натуральных чисел.

Алгоритм НОК с использованием алгоритма НОД:
```c
int NOK (int a, int b)
{
	return a * b / NOD(a, b);
}
```

## Факториал

Нахождение значения факториала:
```c
int n = 3;
int f = 1;
for ( int i = 2; i <= n; i++ )
    f *= i;
cout << "f = " << f << endl;
```

Факториал через рекурсивную функцию:
```c
int fact ( int n )
{
	int f;
	if ( n <= 1 )
		f = 1;
	else f = n * fact( n - 1 );
	return f;
}
```

## Сочетания

Нахождение числа сочетаний:
```c
int C(int n, int k)
{
    return fact(n) / fact(k) / fact(n - k);
}
```


