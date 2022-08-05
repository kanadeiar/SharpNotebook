# Строки, их обработка и символы

## Символы

В .NET символы представлены 16-разрядными кодами стандарта Юникод. Символы - это экземпляр структуры System.Char. 

Для удобства в структуре символа представлено несколько статических методов, облегчающие работу с символами.

Для преобразования символов с учетом региональных стандартов, относящихся к вызывающему потоку методы получают использованием статического свойства CurrentCulture типа System.Globalization.CultureInfo. 

ToString() возвращает односимвольную строку. Методы Parse() & TryParse() получают односимвольную строку и возвращают кодовую позицию UTF-16.

Различные виды преобразований числовых типов в символы и наоборот:

- Приведение типа. Самый эффективный метод.

- Использование типа Convert. Содержит статические методы преобразования из одного типа в другой. В случае потери данных выбрасывается исключение.

- Методы интерфейса IConvertible. Наименее эффективный метод, так как происходит упаковка типов.

Примеры преобразований:

```csharp
char c;
int i;
c = (char)65;
Console.WriteLine(c);
i = (int)c;
c = unchecked((char)(65536 + 65));
Console.WriteLine(i);
Console.WriteLine(c);
c = Convert.ToChar(65);
i = Convert.ToInt32(c);
Console.WriteLine(c);
Console.WriteLine(i);
c = ((IConvertible)65).ToChar(null);
i = ((IConvertible)c).ToInt32(null);
Console.WriteLine(c);
Console.WriteLine(i);
```

## Строки

System.String - неизменяемый упорядоченный набор символов, ссылочный тип, прямой потомок object. Реализует несколько интерфейсво для удобства.

Относится к примитивным типам. Не может создаваться использованием оператора new. Компилятор помещает литеральные строки в метаданные модуля, откуда они загружаются и используются во время выполнения. В CLR объекты string создаются по специальной схеме.

```csharp
string s = "Test string";
```

Для вставки специальных символов, как конец строки, возврат каретки, забой, в C# используются управляющие последовательности. 

```csharp
string s = "Test\r\nstring";
string s = "Test" + Environment.NewLine + "string";
```

Конкатенация строк объединяет литеральные строки в одну на этапе компиляции. Конкатенация других строк - на этапе выполнения.

```csharp
string s = "Test " + "my " + "string";
```

Можно использовать символ буквальных строк с использованием признака буквальных строк @:

```csharp
string path = "C:\\Windows\\System32";
string path = @"C:\Windows\System32";
```

### Неизменяемость строк

Все строки типа string - неизменяемы. Если выполняется много операций над строками, то в куче создается много объектов string. Благодаря неизменяемости отпадает проблема синхронизации потоков при работе со строками. Тип string является запечатанным, тесно интегрирован в среду CLR.

Для того, чтоб уменьшить расход памяти используется интернирование одинаковых строк.

### Сравнение строк

Наиболее часто выполняющаяся операция над строкамия в целях определения равны ли они и для сортировки.

Нужно использовать один из методов, определенных в типе string:

```csharp
bool Equals(string value, StringComparsion comparsionType)
static bool Equals(string a, string b, StringComparsion comparsionType)
static int Compare(string strA, string strB, StringComparsion comparsionType)
static int Compare(string strA, string strB, bool ignoreCase, CultureInfo culture)
static int Compare(string strA, string strB, CultureInfo culture, CompareOptions options)
static int Compare(string strA, int indexA, string strB, int indexB, int length, StringComparsion comparsionType)
static int Compare(string strA, int indexA, string strB, int indexB, int length, bool ignoreCase, CultureInfo culture)
static int Compare(string strA, int indexA, string strB, int indexB, int length, CultureInfo culture, CompareOptions options)
bool StartsWith(string value, StringComparsion comparsionType)
bool StartsWith(string value, bool ignoreCase, CultureInfo culture)
bool EndWith(string value, StringComparsion comparsionType)
bool EndWith(string value, bool ignoreCase, CultureInfo culture)
```

При сортировке следует учитывать регистр символов. В параметре StringComparsion передается тип сравнения.

Когда строки используются внутри программы для решеня внутренних задач, то для таких задач следует использовать флаг Ordinal или OrdinalIgnoreCase. Это самый быстрый способ сравнения.

Когда нужно сравнить строки с учетом лингвистических особенностей, нужно использовать флаг CurrectCulture или CurrentCultureIgnoreCase. Это самый медленный способ сранения.

Сравнение строк в верхнем регистре оптимизировано в фреймворке - используем ToUpperInvariant.

Следует явно указывать, как выполнять сравнение строк, так код будет проще читать.

Объект CultureInfo следует использовать когда нужно использовать региональные стандарты при сортировке строк:

### Интернирование строк

При порядковом сравнении CLR быстро проверяет равенство количества символов, и только если длина одинакова - проводится посимвольное сравнение. При сравнении с учтом региональных стандартов - посимвольно сравнивать даже если длины строк разные.

В CLR поддерживается механизм интернирования строк. При инициализации CLR создает внутреннююю хеш-таблицу, в которой ключами являются строки, а значениями - ссылки на строковые объекты в управляемой куче.

Метод Intern ищет string во внутренней хеш-таблице. При обнарушении возвращается ссылка на объект, иначе - создается копия и добавляется во внутреннюю хеш-таблицу, затем вертается ссылка на объект.

Метод IsInterned получает параметр string и ищет его во внутреннеей хеш-таблице. При удачном поиске - вертает ссылку на объект, иначе - null.

По умолчаию при загрузке сборки CLR интернирует все литеральные строки.
