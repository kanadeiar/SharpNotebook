# Сложности алгоритмов

![Сложность алгоритмов](../img/shpargalka_black.png) 

Сравнение времени выполнения операций нас различными структурами данных, сложность алгоритмов. Времення и объемная (сколько памяти потребуется) сложность.

Структура данных | Временная сложность - 8 колонок  | Объемная сложность
---------------- | ------------------------------------ | ---------------------

Структура          | Среднее |Среднее |Среднее  |Среднее   | Наихудшее |Наихудшее |Наихудшее |Наихудшее | Наихудшее
-------------------|---------|--------|---------|----------|-----------|----------|----------|----------|----------
Название           |Access   |Search  |Insertion|Deletion  |Access     |Search    |Insertion |Deletion  | Объемной
Array              | Θ(1)    | Θ(n)   |Θ(n)     |Θ(n)      |O(1)       |O(n)      |O(n)      |O(n)      |O(n)
Stack              |Θ(n)     |Θ(n)    |Θ(1)     |Θ(1)      |O(n)       |O(n)      |O(1)      |O(1)      |O(n)    
Queue              |Θ(n)     |Θ(n)    |Θ(1)     |Θ(1)      |O(n)       |O(n)      |O(1)      |O(1)      |O(n)    
Single-Linked List |Θ(n)     |Θ(n)    |Θ(1)     |Θ(1)      |O(n)       |O(n)      |O(1)      |O(1)      |O(n)   
Double-Linked List |Θ(n)     |Θ(n)    |Θ(1)     |Θ(1)      |O(n)       |O(n)      |O(1)      |O(1)      |O(n)   
Skip List	   |Θ(log(n))|Θ(log(n))|Θ(log(n))|Θ(log(n))|O(n)       |O(n)      |O(n)      |O(n)      |O(n log(n))   
Hash Table	   |N/A      |Θ(1)    |Θ(1)     |Θ(1)      |N/A        |O(n)      |O(n)      |O(n)      |O(n)   
Binary Search Tree |Θ(log(n))|Θ(log(n))|Θ(log(n))|Θ(log(n))|O(n)       |O(n)      |O(n)      |O(n)      |O(n)   
Castesian Tree     |N/A      |Θ(log(n))|Θ(log(n))|Θ(log(n))|N/A        |O(n)      |O(n)      |O(n)      |O(n)   
B-Tree		   |Θ(log(n))|Θ(log(n))|Θ(log(n))|Θ(log(n))|O(log(n))  |O(log(n)) |O(log(n)) |O(log(n)) |O(n)   
Red-Black Tree     |Θ(log(n))|Θ(log(n))|Θ(log(n))|Θ(log(n))|O(log(n))  |O(log(n)) |O(log(n)) |O(log(n)) |O(n)   
Splay Tree         |N/A      |Θ(log(n))|Θ(log(n))|Θ(log(n))|N/A        |O(log(n)) |O(log(n)) |O(log(n)) |O(n)   
AVL Tree           |Θ(log(n))|Θ(log(n))|Θ(log(n))|Θ(log(n))|O(log(n))  |O(log(n)) |O(log(n)) |O(log(n)) |O(n)   
KD Tree            |Θ(log(n))|Θ(log(n))|Θ(log(n))|Θ(log(n))|O(n)       |O(n)      |O(n)      |O(n)      |O(n)   

Алгоритмы сортировки - временная и объемная (сколько памяти потребуется) сложность.

Алгоритм | Временная сложность - 3 колонки  | Объемная сложность
-------- | -------------------------------- | -------------------

Алгоритм           | Лучшее     |Среднее    | Худшее     |Худшее      
-------------------|------------|-----------|------------|------------
Quicksort          |Ω(n log(n)) |Θ(n log(n))|O(n^2)      |O(log(n))    
Mergesort 	   |Ω(n log(n)) |Θ(n log(n))|O(n log(n)) |O(n)
Timsort 	   |Ω(n) 	|Θ(n log(n))|O(n log(n)) |O(n)
Heapsort 	   |Ω(n log(n)) |Θ(n log(n))|O(n log(n)) |O(1)
Bubble Sort 	   |Ω(n) 	|Θ(n^2)     |O(n^2) 	 |O(1)
Insertion Sort 	   |Ω(n) 	|Θ(n^2)     |O(n^2) 	 |O(1)
Selection Sort 	   |Ω(n^2) 	|Θ(n^2)     |O(n^2) 	 |O(1)
Tree Sort 	   |Ω(n log(n)) |Θ(n log(n))|O(n^2) 	 |O(n)
Shell Sort 	   |Ω(n log(n)) |Θ(n(log(n))^2)|O(n(log(n))^2)|O(1)
Bucket Sort 	   |Ω(n+k) 	|Θ(n+k)     |O(n^2) 	 |O(n)
Radix Sort 	   |Ω(nk) 	|Θ(nk) 	    |O(nk) 	 |O(n+k)
Counting Sort 	   |Ω(n+k) 	|Θ(n+k)     |O(n+k) 	 |O(k)
Cubesort 	   |Ω(n) 	|Θ(n log(n))|O(n log(n)) |O(n)


Источник: [Большой алгоритмический О-лист](https://www.bigocheatsheet.com/).

