# Функции (аргументы и вертаемое значение)

## Определение функций

Разделение кода на функции позволяет сосредоточится на том участке кода, который требуется изменить, локализовать его. Программы на языке С состоят из множества функций. Свою программу если она делает множество операций можно разделить на более меньшие логически обоснованные куски.

Разбиение программы на более мелкие функции называется структурным программированием.

Пример разбиения программы на подфункции:
```c
void assignID()
{
	//присваивание ид новому сотруднику
}
void buildContact()
{
	//построение контакта
}
void payAdd()
{
	//доабвение сотрудника нового в список
}
int main(int argc, const char* argv[])
{
	SetConsoleCP(1251);
	SetConsoleOutputCP(1251);
	assignID();
	buildContact();
	payAdd();
	return 0;
}
```

Любая функция может вызывать любую другую функцию. 

Переменная считается локальной если она определяется после открывающей фигурной скобки функции. Перемення объявленная внутри усатых скобок функции является локальной. Локальная переменная безопаснее глобальной. Локальные переменные могут использоватся только внутри блока кода, где они объявлены.
 
Переменная считается глобальной только если эта переменная объявляется перед имененм функции. Любая функция может изменять глобальную переменну, значит она не безопасна.

Для упрощения поддержки и ускорения разработки следует разбивать как можно чаще код на отдельные функции.

## Аргументы функции

Не нужно, чтобы все функции имели доступ ко всем глобальным переменным. Для совместного использования данных разными функциями, нужно передавать переменные от функции к функции. Когда одна функция передает другой функции переменную, только они обе имеют к ней доступ. Для передачи переменных используют аргументы функции. Принимающая функция принимает параметры от функции, передающей переменные.

Переменные можно передавать двумя способами: по значению и по адресу. Переменные, которые нужно передать, помещаются внутрь скобок на принимающей и передающей стороне.

Передача по значению можно называть передачей по копии. Передается только значение, но не сама переменная.

Пример передачи по значению:
```c
int half (int i); //Прототип функции
int main(int argc, const char* argv[])
{
	SetConsoleCP(1251);
	SetConsoleOutputCP(1251);
	/////////////////////////////////////////////////////
	puts("Введите целое число:> ");
	int i;
	scanf_s(" %d", &i);
	int d = half(i);
	printf_s("Значение введенное: %d измененное: %d\n", i, d);
	return 0;
}
int half (int i)
{
	i = i / 2; //изменение переданной копии значения
	return i;
}
```
Все переменные не массивы передаются по значению. Функции передаются только копии этой простой переменной. В вызывающей строне переменные не меняются при изменении в локальной функции.

При передаче массива в другую функцию, этот массив передаетя по адресу. Вместо передачи копии массива принимающей функции передается только копия адреса массива. Принимающая функция записывает адрес в переменную-указатель массива. И вызывающая и принимающая строны фактически работают с одним и тем-же массивом. Если принимающая функция изменит массив, то в вызывающей он также изменится.

Пример передачи массива в функцию:
```c
void change (char name[9]); //Прототип функции
int main(int argc, const char* argv[])
{
	SetConsoleCP(1251);
	SetConsoleOutputCP(1251);
	/////////////////////////////////////////////////////
	char name[9] = "Agnat";
	change(name);
	printf_s("Измененное имя: %s\n", name);
	return 0;
}
void change (char name[9])
{
	strcpy_s(name, 9, "********");
}
```

Можно передавать обычную переменную по адресу:
```c
void half (int * i); //Прототип функции
int main(int argc, const char* argv[])
{
	SetConsoleCP(1251);
	SetConsoleOutputCP(1251);
	/////////////////////////////////////////////////////
	puts("Введите целое число:> ");
	int i;
	scanf_s(" %d", &i);
	int d = i;
	half(&d);
	printf_s("Значение введенное: %d измененное: %d\n", i, d);
	return 0;
}
void half (int * i)
{
	*i = *i / 2;
}
```
Чтобы защитить переменные от изменения вызываемой функцией, нужно передавтаь их по значению. Но если нужно, чтобы она их изменяла, то нужно передавать по адресу.

Если нужно передать переменную - не массив по адресу, то нужно перед переменной ставить амперсанд &.

## Вертаемое значение из функции

Для того, чтобы вернуть значение из функции, нужно использовать выражение return, заключая в скобки вертаемое значение. Если функция ниче не возвращает, выражение return можно не ставить.

Пример примененеия функции с возвращаемым значением:
```c
int intSum (int i, int j, int k); //Прототип функции
int main(int argc, const char* argv[])
{
	SetConsoleCP(1251);
	SetConsoleOutputCP(1251);
	/////////////////////////////////////////////////////
	int a = 5;
	int b = 8;
	int c = 11;
	int result = intSum(a,b,c);
	printf_s("Сумма значений (%d, %d, %d): %d\n", a, b, c, result);
	return 0;
}
int intSum (int i, int j, int k)
{
	return i + j + k;
}
```

## Прототип функции

Прототип функции - это модель настоящей функции, контакт. Пример модели или прототипа:

int aFunc(int x, int y);

Для создания прототипа нужно поместить точную копию первой строки функции где-нибудь перед main(). Эта строка не является вызовом функции, так как она находится перед функцией main(). Эта строка - прототип функции.

Протипы стандартных функций вроде printf() и putc() находятся в подключемых заголовочных файлах. Прототип фунцкии сообщает компилятору, что ему ожидать в дальнейшем ниже в коде.
