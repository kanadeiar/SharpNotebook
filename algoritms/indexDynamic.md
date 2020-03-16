# Динамическое программирование

## Нахождение числа Фибоначчи

Пример нахождения числа с использованием руекрсивного вызова функции:
```c
#define MAXN 45 //наибольшее представляющее интерес число n
#define UNKNOWN -1 //пустая ячейка
long f[MAXN+1]; //массив для хранения вычисленных значений fib
long fib_c(int n)
{
	if (f[n] == UNKNOWN)
		f[n] = fib_c(n-1) + fib_c(n-2);
	return f[n];
}
long fib_c_driver(int n)
{
	int i;
	f[0]=0;
	f[1]=1;
	for (i=2; i<=n; i++)
		f[i]=UNKNOWN;
	return fib_c(n);
}
	long num = fib_c_driver(6);
	printf("val = %d\n", num);
```
Пример нахождения посредством динамического программирования:
```c
long fib_u(int n)
{
	int i;
	long back2=0;
	long back1=1;
	long next;
	if (n==0) return 0;
	for (i=2; i<=n; i++)
	{
		next=back1+back2;
		back2=back1;
		back1=next;
	}
	return back1;
}
	long n = fib_u(6);
	printf("val = %d\n", n);
```


