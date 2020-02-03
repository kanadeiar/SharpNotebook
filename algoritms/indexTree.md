# Структуры - деревья

## Простое дерево:

```c
typedef int T;
//узел дерева
typedef struct Node
{
	T data;
	struct Node * left;
	struct Node * right;
} Node;
//распечатка дерева
void printTree(Node * root)
{
	if (root)
	{
		printf("%d", root->data);
		if (root->left || root->right)
		{
			printf("(");
			if (root->left)
				printTree(root->left);
			else
				printf("NULL");
			printf(",");
			if (root->right)
				printTree(root->right);
			else
				printf("NULL");
			printf(")");
		}
	}
}
FILE * file;
//создание сбалансированного дерева по файлу
Node * Tree(int n)
{
	Node * newNode;
	if (n==0)
		newNode = NULL;
	else
	{
		int x;
		fscanf(file, "%d", &x);
		int nl = n / 2;
		int nr = n - nl - 1;
		newNode = (Node *) malloc(sizeof(Node));
		newNode->data = x;
		newNode->left = Tree(nl);
		newNode->right = Tree(nr);
	}
	return newNode;
}
//использование:
	fopen_s(&file,"c:\\Temp\\digitals.txt", "r");
	if (file == NULL)
	{
		puts("Ошибка отрытия файла");
		return 1;
	}
	Node * tree = NULL;
	int count;
	fscanf(file, "%d", &count);
	tree = Tree(count);
	fclose(file);
	printTree(tree);
```

## Дерево двоичного поиска

Пример:
```c
#define T int
/**
 * \brief Узел дерева
 */
typedef struct Node
{
	T value;
	struct Node * left;
	struct Node * right;
	struct Node * parent;
} Node;
/**
 * \brief Печать дерева в консоль
 * \param root дерево
 */
void PrintTree(Node * root)
{
	if (root)
	{
		printf("%d", root->value);
		if (root->left || root->right)
		{
			printf("(");
			if (root->left)
				PrintTree(root->left);
			else
				printf("NULL");
			printf(",");
			if (root->right)
				PrintTree(root->right);
			else
				printf("NULL");
			printf(")");
		}
	}
}
/**
 * \brief Создание нового пустого узла
 * \param value значение
 * \param parent родитель
 * \return новый узел
 */
Node * GetNewNode(T value, Node * parent)
{
	Node *tmp = (Node *) malloc(sizeof(Node));
	tmp->left = tmp->right = NULL;
	tmp->value = value;
	tmp->parent = parent;
	return tmp;
}
/**
 * \brief Вставка нового элемента в дерево
 * \param head голова дерева
 * \param value значение
 */
void InsertNode(Node **head, int value)
{
	Node *tmp = NULL;
	if (*head == NULL)
	{
		*head = GetNewNode(value, NULL);
		return;
	}
	tmp = *head;
	while (tmp)
	{
		if (value > tmp->value)
		{
			if (tmp->right)
			{
				tmp = tmp->right;
				continue;
			}
			else
			{
				tmp->right = GetNewNode(value, tmp);
				return;
			}
		}
		else if (value < tmp->value)
		{
			if (tmp->left)
			{
				tmp = tmp->left;
				continue;
			}
			else
			{
				tmp->left = GetNewNode(value, tmp);
				return;
			}
		}
		else
		{
			exit(2);
		}
	}
}
//использование:
	Node *Tree = NULL;
	FILE *file = fopen("digitals.txt", "r");
	if (file == NULL)
	{
		puts("Невозможно открыть файл для чтения");
		getchar();
		exit(1);
	}
	int count;
	fscanf(file, "%d", &count);
	int i;
	for (i = 0; i<count; i++)
	{
		int value;
		fscanf(file, "%d", &value);
		InsertNode(&Tree, value);
	}
	fclose(file);
	PrintTree(Tree);
	printf("\nЕще: ");
	PreOrderTravers(Tree);
```
Обход элементов различными способами:
```c
/**
 * \brief прямой обход дерева КЛП "корень-левый-правый"
 * \param root голова дерева
 */
void PreOrderTravers(Node *root)
{
	if (root)
	{
		printf("%d ", root->value);
		PreOrderTravers(root->left);
		PreOrderTravers(root->right);
	}
}
/**
 * \brief симметричный обход ЛКП - "левый-корень-правый"
 * \param root голова дерева
 */
void SimmOrderTravers(Node *root)
{
	if (root)
	{
		SimmOrderTravers(root->left);
		printf("%d ", root->value);
		SimmOrderTravers(root->right);
	}
}
/**
 * \brief обратный обход ЛПК - "левый-правый-корень"
 * \param root голова дерева
 */
void AfterOrderTravers(Node *root)
{
	if (root)
	{
		AfterOrderTravers(root->left);
		AfterOrderTravers(root->right);
		printf("%d ", root->value);
	}
}
```
Поиск элемента в дереве:
```c
/**
 * \brief поиск элемента
 * \param root голова дерева
 * \param value значение
 * \return элемент найденный
 */
Node *FindInTree(Node *root, T value)
{
	if (root)
	{
		if (root->value == value)
			return root;
		Node *tmpleft = FindInTree(root->left, value);
		if (tmpleft != NULL)
			return tmpleft;
		Node *tmpright = FindInTree(root->right, value);
		if (tmpright != NULL)
			return tmpright;
	}
	return NULL;
}
```


