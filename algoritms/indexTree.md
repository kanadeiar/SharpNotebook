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


