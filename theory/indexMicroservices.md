# Микросервисы

Архитектурный стиль микросервисов — это подход, при котором единое приложение строится как набор небольших сервисов, каждый из которых работает в собственном процессе и коммуницирует с остальными используя легковесные механизмы, как правило HTTP. Эти сервисы построены вокруг бизнес-потребностей и развертываются независимо с использованием полностью автоматизированной среды. 

Существует абсолютный минимум централизованного управления этими сервисами. Сами по себе эти сервисы могут быть написаны на разных языках и использовать разные технологии хранения данных.

Микросервисная архитектура — это подход к созданию приложения, подразумевающий отказ от единой, монолитной структуры.

Преимущества

Монолит | Микросервисы
------------ | -------------
Простота | Частичное развертывание
Согласованность | Доступность
Межмодульный рефакторинг | Сохранение модульности
| | Мультиплатформенность