# Введение в платформу .NET

10 ноября 2020 года Микрософты выпустили C# 9 & .NET 5. Как и C# 8, C# 9 связан с версией фреймворка и платформы и будет работать на .NET 5 и выше. .NET 5 - продолжение .NET Core.

.NET Core позволяет разрабатывать веб приложения и сервисы на Windows, iOS & Linux, мобильные приложения на Android & iOS, а также настольные приложения только на Windows.

## Общеязыковая среда исполнения Core Runtime

Формально можно разделить на CoreCLR & CoreFX. С точки зрения прогера .NET Core - это общеязыковая среда исполнения и всеоблеющая библиотека базовых классов. 

Слой исполнения содержит минимальную реализацию, специфичную для платформы (Windows, iOS, Linux, Android) и архитектуры (x86, x64, ARM), использует все базовые типы .NET Core.

## Общая система типов CTS

Спецификация Common Type System (CTS) описывает все типы и все программные конструкции, поддерживаемые средой исполнения, спецификации их работы, описанные в виде метаданных.

Поле - переменная, являющаяся частью состояния объекта.

Метод - функция, выполняющая операцию с объектом, часто с изменением его состояния.

Свойство - выглядить как поле, но в реализации типа является методом (или двумя методами).

Событие - для создания меанизма оповещения между объектом и другими заинтересованными объектами.

Класс, основан на парадигме ООП, основа - ссылочный тип, содержит: конструкторы, свойства, методы, события и поля.

Интерфейс - именованная коллекция абстрактных членов (и/или реализаций по умолчанию) класса.

Структура - легковесная реализация класса, основа - значимый тип, содержит контструкторы, методы и поля.

Перечисление - удобная структура комбинаций ключ-значение на основе значимого типа.

Делегат - типобезопасный эквивалент указателя на функцию языка С. Служит для реализации функции обртного вызова.

## Общеязыковая спецификация CLS

Так как не все языки платформы .NET Core поддерживают все что есть в общей системе типов, есть спецификация CLS, в которой описаны типы, которые должен поддерживать всякий язык платфонмы. 

Это подмножество, описанное в общей системе типов CTS.

Спецификация CLS - набор подробных правил, описывающих минимальное и полное множество характеристик, который отдельный компилятор .NET должен поддерживать, чтобы генерировать код, обслуживаемый средой CLR и в тоже время доступный другим языкам, также ориентированных на эту платформу. Можно рассматривать общеязыковую спецификацию CLS как подмножество полной функциональнотси, определенной в общей системе типов CTS.

Это набор правил, которых должны придерживатся создатели компиляторов, если они намерены обеспечивать работу своих сборок в платформе .NET. 

Сообщить компилятору о том, что он должен сделать проверку на совместимость с CLS в проекте:

```csharp
[assembly: CLSCompliant(true)]
```

## Библиотека базовых классов

Платформа содержит библиотеку базовых классов BCL, которая доступная на всех языках платформы. Она описывает типы, которые могут быть использованы для разработки любого программного обеспечения и компонентов.

## Стандартная инфраструктура разработки библиотек

.NET Standard описывает типы, которые поддерживаются как .NET Core, так и старой платформой .NET Framework. 

## Управляемый и неуправляемый код

Код, написанный и нацеленный на исполнение в среде .NET Core, является управляемым. Код, не выполняемый в среде .NET Core, является неуправляемым. Оба кода могут быть взяимосвязаны.

Двоичные модули, .NET внутренне устроены совершенно отлично от неуправляемых (*.dll и *.exe). Они содержат не специфические, а независимые от платформы инструкции на промежуточном языке и метаданные типов. В отличие от старой платформы .NET Framework, в новой .NET Core всегда компилируются только файлы с расширением *.dll, в том числе и выполняемые.



