# Алгоритмы C# основы

	setlocale(LC_ALL, "russian");
    getchar();
	return 0;
    
    #include <windows.h>
	SetConsoleCP(1251);
    SetConsoleOutputCP(1251);
    
    
    char name[64];
	if (scanf_s(" %s", name, sizeof(name)) == 1)
		printf("Your name is %s\n", name);