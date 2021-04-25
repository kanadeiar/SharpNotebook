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

