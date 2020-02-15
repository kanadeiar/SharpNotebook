# Структуры - связные списки

Каждый узел в структуре данных содержит одно или несколько полей для хранения данных.

Каждый узел содержит поле указателя на следующий узел.

Требуется указатель на начало структуры, чтоб знать откуда начинать обращение к ней.

## Односвязный список

Объявление структуры:
```c
typedef struct list
{
	int item;
	struct List * next;
} List;
```

Вставка элемента в связный список:
```c
void Indert(List **l, int x)
{
	List * tmp;
	tmp = malloc(sizeof(List));
	tmp->item = x;
	tmp->next = *l;
	*l = tmp; //вставка элемента в голову списка
}
```

Поиск элемента в связном списке:
```c
List * SearchList(List *l, int x)
{
	if (l == NULL)
		return 0;
	if (l->item == x)
		return l;
	else
		SearchList(l->next, x);
}
```

Удаление элемента из связного списка:
```c
List * SearchPreList(List *l, int x)
{
	if (l == NULL || l->next == NULL)
		return NULL;
	List * tmp = l->next;
	if (tmp->item == x)
		return l;
	else
		return SearchPreList(l->next, x);
}
void DeleteList(List **l, int x)
{
	List * delMe; //удаляемый узел
	List * pre; //предидущий узел
	delMe = SearchList(*l, x);
	if (delMe != NULL)
	{
		pre = SearchPreList(*l, x);
		if (pre == NULL)
			*l = delMe->next;
		else
			pre->next = delMe->next;
		free(delMe);
	}
}
```

Вывод значений связного списка:
```c
void PrintList(List *l)
{
	if (!l)
		return;
	printf("%d\n", l->item);
	PrintList(l->next);
}
```

Использование:
```c
List * head = NULL;
InsertList(&head, 3);
InsertList(&head, 2);
InsertList(&head, 1);
puts("Один элемент:");
List * s = SearchList(head, 1);
printf("%d\n", s->item);
puts("Массив:");
PrintList(head);
DeleteList(&head, 1);
DeleteList(&head, 2);
DeleteList(&head, 3);
puts("Массив:");
PrintList(head);
```

## Двусвязный список

Объявление структуры:
```c
typedef struct dlist
{
	int item;
	struct DList * next;
	struct DList * prev;
} DList;
```
Вставка элемента в двусвязный список в голову либо в хвост:
```c
void InsertHDList(DList **h, DList ** t, int x)
{
	DList * tmp;
	tmp = malloc(sizeof(DList));
	tmp->item = x;
	tmp->next = *h;
	tmp->prev = NULL;
	if (*h != NULL)
		(*h)->prev = tmp;
	else
		*t = tmp;
	*h = tmp; //вставка элемента в голову списка
}
void InsertTDList(DList **h, DList **t, int x)
{
	DList * tmp;
	tmp = malloc(sizeof(DList));
	tmp->item = x;
	tmp->next = NULL;
	tmp->prev = *t;
	if (*t != NULL)
		(*t)->next = tmp;
	else
		*h = tmp;
	*t = tmp; //вставка элемента в хвост списка
}
```
Поиск элемента в двусвязном списке:
```c
DList * SearchHeadDList(DList *h, DList *t, int x)
{
	if (h == NULL)
		return 0;
	if (h->item == x)
		return h;
	else
		SearchHeadDList(h->next, t, x);
}
DList * SearchTailDList(DList *h, DList *t, int x)
{
	if (t == NULL)
		return 0;
	if (t->item == x)
		return t;
	else
		SearchTailDList(h, t->prev, x);
}
```
Удаление элемента из двусвязного списка:
```c
DList * SearchHDPreList(DList *h, DList *t, int x)
{
	if (h == NULL || h->next == NULL)
		return NULL;
	DList * tmp = h->next;
	if (tmp->item == x)
		return h;
	else
		return SearchHDPreList(h->next, t, x);
}
DList * SearchHDNextList(DList *h, DList *t, int x)
{
	if (h == NULL || h->next == NULL)
		return NULL;
	DList * tmp = t->next;
	if (h->item == x)
		return tmp;
	else
		return SearchHDNextList(h->next, t, x);
}
void DeleteHeadList(DList **h, DList **t, int x)
{
	DList * delMe; //удаляемый узел
	DList * pre; //предидущий узел
	DList * next; //следующий узел
	delMe = SearchHeadDList(*h, *t, x);
	if (delMe != NULL)
	{
		pre = SearchHDPreList(*h, *t, x);
		next = SearchHDNextList(*h, *t, x);
		if (pre == NULL)
			*h = delMe->next;
		else
			pre->next = delMe->next;
		if (next == NULL)
			*t = delMe->prev;
		else
			next->prev = delMe->prev;
		free(delMe);
	}
}
```
Использование:
```c
DList * head = NULL;
DList * tail = NULL;
InsertHeadDList(&head, &tail, 1);
InsertHeadDList(&head, &tail, 2);
InsertHeadDList(&head, &tail, 3);
puts("Один элемент:");
DList * s = SearchHeadDList(head, tail, 4);
printf("%d\n", s->item);
puts("Один элемент:");
s = SearchTailDList(head, tail, 1);
printf("%d\n", s->item);
puts("Массив до удаления:");
PrintHDList(head, tail);
DeleteHeadList(&head, &tail, 1);
DeleteHeadList(&head, &tail, 2);
DeleteHeadList(&head, &tail, 3);
puts("Массив после удаления:");
PrintHDList(head, tail);
```





