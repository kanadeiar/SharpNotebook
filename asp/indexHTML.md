# Веб HTML+CSS

## Настройка среды разработки Visual Studio Code

WordWrap : column

WordWrapColumn : 120

FontSize : 12

FormatOnSave : on

Расширения:

Русификатор

Рыбный текст на русском

Препроцессор Sass

Компилятор Live Sass Compiler

## Основные теги

### Заголовки

```html
<h1>Главный заголовок</h1>
```

### Параграфы текста

```html
<p>Параграф</p>
```
Рыбатекст - lorem<tab>

Выделение текста:

```html
<strong> и <b> - выделение текста полужирным
    
<em> и <i> - выделение текста курсивом
    
<small> - уменьшает размер шрифта на еденицу
    
<sub> - нижний индекс
    
<sup> - верхний индекс
    
<ins> - подчеркивание текста
    
<del> - перечеркивание текста
    
<code> - вставка кода
    
<pre> - вывод текста в исходном форматировании

<q> - выделение цитат, заклюение в кавычки
```

Вложение тегов:
```html
<p>Параграф 
    <b>жирный 
        <i>и курсив</i>
    </b>
</p>
```

### Спецсимволы

HTML | Внешний вид | Описание
------------ | ------------- | --------------
&nbsp |  | Неразрывный пробел
&copy | © | Знак авторского права
&quot; и &raquo; | « и » | Двойные кавычки
&lt; и &gt; | < и > | Символы
&prime; | ' | одинарная кавычка

### Списки

Маркированный список:
```html
<ul>
    <li>Первый элемент списка</li>
    <li>Второй элемент списка</li>
    <li>Третий элемент списка</li>
</ul>
```

Нумерованный список:
```html
<ol>
    <li>Первый элемент списка</li>
    <li>Второй элемент списка</li>
    <li>Третий элемент списка</li>
</ol>
```

Многоуровневый список:
```html
<ol>
    <li>Первый элемент списка</li>
    <li>Второй элемент списка
        <ol>
           <li>Первый элемент вложенного списка</li>
           <li>Второй элемент вложенного списка</li>
           <li>Третий элемент вложенного списка</li>
        </ol>
    </li>
    <li>Третий элемент списка</li>
</ol>
```

### Гиперссылки

Относительные ссылки
```html
<a href="file_name.html">текст ссылки, который видит пользователь</a> 
```
Абсолютные ссылки
```html
<a href="https://mail.ru" target="_blank">
    страница mail.ru откроется в новой вкладке
</a> 
```
Якоря
```html
<a href="#p10">Прочитать 10 параграф</a>
<p id="p1">Текст параграфа №1</p>
<p id="p2">Текст параграфа №2</p>
...
<p id="p10">Вот и долгожданный параграф №10</p>
```

### Изображения
```html
<img src="img/my_foto.jpg" alt="это моя фотография" title="это моя фотография" width="300">
```

### Формы
Ввод информации различных типов:
```html
<input type="text" size="30" placeholder="Ваше Имя">
<input type="password" size="8">
<input type="checkbox" checked="checked">
<input type="checkbox" checked>
<input type="radio" name="radio">
<input type="file">
<input type="submit" value="Сохранить">
<input type="reset" value="Очистить">
<input type="button" value="просто кнопка">
<button>Кнопка html5</button>
```
Многострочное поле ввода информации:
```html
<textarea rows="10" placeholder="Введите текст сюда">
```
Метка - связь текта с элементом формы:
```html
<!-- Первый способ -->
<input id="идентификатор" type="checkbox">
<label for="идентификатор">Текст</label>
<!-- Второй способ -->
<label><input type="checkbox">Текст</label>
```

## Каскадная таблица стилей CSS



## Базовая разметка простого сайта

Разметка:
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="css/style.css">
    <title>Document</title>
    <link rel="icon" href="favicon.ico" type="image/x-icon">
    <link rel="shortcut icon" href="favicon.ico" type="image/x-icon">
</head>

<body>
    <div class="wrap">
        <div class="wrap_top">
            <header class="header">
                <div class="container">
                    <a class="header_link" href="#">Название сайта</a>
                </div>
            </header>
            <nav class="nav container">
                <ul class="menu">
                    <li class="menu_list">
                        <a href="" class="menu_link">ссылка-1</a>
                    </li>
                    <li class="menu_list">
                        <a href="" class="menu_link">ссылка-2</a>
                    </li>
                    <li class="menu_list">
                        <a href="" class="menu_link">ссылка-3</a>
                    </li>
                </ul>
            </nav>
            <main class="content container">
                <div class="content_item">
                    <p>Не вызывает сомнений, что новая модель организационной деятельности требует от нас анализа
                        поэтапного и последовательного развития общества.</p>
                </div>
                <div class="content_item">
                    <p>Для.</p>
                </div>
                <div class="content_item">
                    <p>Повседневная.</p>
                </div>
            </main>
        </div>
        <footer class="footer">
            <div class="container">
                <h3>Подвал</h3>
                <div class="footer_copyright">
                <p>&copy; 2021 Все права защищены</p>
            </div>
            </div>
        </footer>
    </div>
</body>

</html>
```

SCSS:
```css
$width_site: 1140px;
$color-site: #afa;

* {
    margin: 0;
    padding: 0;
}

.container {
    max-width: $width_site;
    margin: 0 auto;
}

body {
    font-family: sans-serif;
    font-size: 16px;
    line-height: 26px;
    color: #444;
}

/* Прижатие подвала к полу */
.wrap {
    min-height: 100vh;
    display: flex;
    flex-direction: column;
    &_top {
        flex-grow: 1;
    }
}

/* Заголовок */
.header {
    height: 100px;
    background-color: $color-site + #222;
    .container {
        padding-top: 40px;
    }
    &_link {
        font-size: 40px;
        text-decoration: none;
        color: black;
    }
}

/* Тело */
.nav {
    background-color: $color-site + #333;
    .menu {
        display: flex;
        justify-content: start;
        &_list {
            list-style-type: none;
        }
        &_link {
            margin: 10px 1px;
            padding: 10px 5px;
            text-decoration: none;
            display: block;
        }
    }
}
.content {
    background-color: $color-site + #111;
    display: flex;
    flex-wrap: wrap;
    justify-content: space-between;
    &_item {
        background-color: $color-site - #222;
        width: $width_site / 3;
        height: 200px;
        margin: 10px 0;
        box-sizing: border-box;
        border: 1px solid #111;
        padding: 10px;
    }
}

/* Подвал */
.footer {
    height: 100px;
    background-color: $color-site + #222;
}

```

