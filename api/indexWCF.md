# Основы WCF

Инфраструктура WCF является инструментальным набором для распределенных вычислений, интегрирующей все ранее независимые технологии распределенной обработки в один рационализированный API-интерфейс, главным образом из спространства имен System.ServiceModel. 

WCF позволяет открыть доступ к службам для вызывающих компонентов широким многообразием приемов. WPF позволяет выбирать корректный протокол для выполнения работы, подключать связующий код распределенного приложения становится легко. Утомительные детали могут переносится в конфигурационные файлы приложения.

Главные средства WCF:

- Поддержка строго типизированных и нетипизированных сообщений.

- Поддержка нескольких привязок.

- Поддержка последних спецификаций веб-служб (WS-*).

- Интегрированная модель безопасности, охватывающая и собственные и многочисленные нейтральные технологии защиты на основе веб-служб.

- Поддержка технологий сеансового управления состоянием и технологий однонаправленных сообщений без состояния.

- Средста трассировки и протоколирования.

- Счетчики производительности.

- Модель публикации и подписки на события.

- Поддержка транзакций.

Некоторые сборки WCF:

Сборка                           | Описание
---------------------------------|-------------------------
System.Runtime.Serialization.dll | Пространства имен и типы, используемые для сериализации и десериализации объектов в инфраструктуре WCF
System.ServiceModel.dll          | Типы, определяемые для построения приложения WCF любого типа

Некоторые пространства имен WCF:

Сборка                           | Описание
---------------------------------|-------------------------
System.Runtime.Serialization     | Многочисленные типы, используемые для сериализации и десериализации объектов в инфраструктуре WCF
System.ServiceModel              | 

## Архитектура, ориентированная на службы

Это служба (SOA) - способ проектирования распределенной системы, где несколько автономных служб работают совместно, передавая сообщения через границы - сетевых машин или двух процессов на одной машине через четко определенные интерфейсы. Интерфейсы служб описывают набор членов, которые могут быть вызваны внешними компонентами.

Принципы SOA:

- **Границы устанавливаются явно.** Функциональность службы выражается в виде четко определенных интерфейсов, внешний компонент остается в неведении относительно деталей внутренней реализации.

- **Службы являются автономными.** Каждая служба является отдельным "островком". Должна быть независима в отношении версии, развертывания и установки. После того как интерфейс передан в производственную среду, он не меняется, только лишь дополняется новыми интерфейсами, моделирующими желаемую функциональность.

- **Службы взаимодействуют через контракт.** Детали реализации службы не касаются вызывающего внешнего компонента, взаимодействие только через открытые внешние интерфейсы.

- **Совместимость служб на основе политики.** Интерфейсы и WSDL (язык описания веб-служб) недостаточно выразительны относительно того, что должна делать служба, и SOA позволяет определять политики, дополнительно проясняющие семантику службы, описывающие низкоуровневое описание службы. 





