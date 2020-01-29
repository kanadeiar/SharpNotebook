# Структуры простые

## Стек

Пример на массиве:
```c
T Stack[MaxN];
int n = -1;
void push(T i)
{
	if (n < MaxN)
	{
		n++;
		Stack[n] = i;
	}
	else
		printf("Стек переполнен");
}
T pop()
{
	if (n != -1)
		return Stack[n--];
	else
		printf("Стек пуст");
}
```
Пример на списках:
```c
struct TNode
{
	int value;
	struct TNode * next;
};
typedef struct TNode Node;
struct Stack
{
	Node * head;
	int size;
	int maxSize;
};
struct Stack stack;
void push(int value)
{
	if (stack.size >= stack.maxSize)
	{
		puts("Переполнение стека");
		return;
	}
	Node * tmp = (Node*) malloc(sizeof(Node));
	tmp->value = value;
	tmp->next = stack.head;
	stack.head = tmp;
	stack.size++;
}
int pop()
{
	if (stack.size == 0)
	{
		puts("Стек пуст!");
		return -1;
	}
	Node * deleteme = stack.head;
	int value = stack.head->value;
	stack.head = stack.head->next;
	free(deleteme);
	stack.size--;
	return value;
}
int print_stack()
{
	Node * pI = stack.head;
	while (pI != NULL)
	{
		printf("%d ", pI->value);
		pI = pI->next;
	}
}
	stack.maxSize = 100;
	stack.head = NULL;
	push(1);
	push(2);
	push(3);
	print_stack();
```




## Очередь

Пример на массиве:
```c
T Queue[MaxN];
int front = 0;
int rear = -1;
int itemCount = 0;
void enqueue(T i)
{
	if (itemCount != MaxN)
	{
		if (rear == MaxN-1)
			rear = -1;
		Queue[++rear] = i;
		itemCount++;
	}
	else
		printf("Очередь переполнена");
}
T dequeue()
{
	T temp = Queue[front++];
	if (front == MaxN)
		front = 0;
	itemCount--;
	return temp;
}
T peek()
{
	return Queue[front];
}
```

## Дек

