# Общеязыковая спецификация CLS

Общеязыковая спецификация CLS (Common Language specification)описывает подмножество общих типов и программных коснтрукций, которые должны поддерживать все языки программирования среды .NET. 

Язаки платформы несмотря что компилятором преборазуются в схожий общий промежуточный код CIL, могут отличатся в отношении общего уровня функциональности. При таких обстоятельствах необходимы общие правила для всех языков, ориентированных на .NET.

Спецификация CLS - набор подробных правил, описывающих минимальное и полное множество характеристик, который отдельный компилятор .NET должен поддерживать, чтобы генерировать код, обслуживаемый средой CLR и в тоже время доступный другим языкам, также ориентированных на эту платформу. Можно рассматривать общеязыковую спецификацию CLS как подмножество полной функциональнотси, определенной в [общей системе типов CTS](./cts.md).

Это набор правил, которых должны придерживатся создатели компиляторов, если они намерены обеспечивать работу своих сборок в [платформе .NET](./index.md). 

Сообщить компилятору о том, что он должен сделать проверку на совместимость с CLS в проекте:
```csharp
[assembly: CLSCompliant(true)]
```

Каждый язык программирования .NET должен поддерживать эту спецификацию общих типов и программных конструкций. 