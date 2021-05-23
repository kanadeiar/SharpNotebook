# Веб HTML+CSS Адаптив


## Базовая разметка простого адаптивного сайта

Разметка:
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="css/style.css">
    <link rel="icon" href="favicon.ico" type="image/x-icon" />
    <link rel="shortcut icon" href="favicon.ico" type="image/x-icon" />
    <title>Заголовок сайта</title>
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
            </div>
        </footer>
    </div>

</body>

</html>
```

SCSS:
```css
$widthSite: 1024px;
$color-site: #afa;

* {
    margin: 0;
    padding: 0;
}

.container {
    max-width: $widthSite;
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
    &_link {
        padding: 10px 5px;
        text-decoration: none;
        display: block;
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
        width: $widthSite / 3;
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
    h3 {
        padding: 10px 5px;
    }
}

/* Адаптивный планшетный дизайн */
@media (max-width:1024px) {
    .container {
        padding-left: 32px;
        padding-right: 32px;
    }
    .header {
        height: 150px;
        &_link {
            padding: 30px 5px;
        }
    }
    .content {
        justify-content: center;
        &_item {
            width: 50%;
        }
    }
    .footer {
        height: 150px;
    }
}

/* Адаптивный мобильный дизайн */
@media (max-width: 667px) {
    .container {
        padding-left: 20px;
        padding-right: 20px;
    }
    .header {
        min-height: 400px;
        box-sizing: border-box;
        .container {
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        &_link {
            padding: 60px 5px;
        }
    }
    .content {
        justify-content: space-between;
        &_item {
            width: 100%;
        }
    }
    .footer {
        height: 200px;
    }
}
```




