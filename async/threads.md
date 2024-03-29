## Потоки

Каждый экземпляр приложения запускается в отдельном процессе. Процессом называется набор ресурсов, используемый отдельным экземпляром приложения. Каждому процессу выделяется виртуальное адресное пространство. Это гарантирует, что код и данные одного процесса будут недоступны для другого.

Каждому Windows-процессу выделяется собственный поток исполнения (который работает как виртуальный процессор). Если код приложения войдет в бесконечный цикл, то блокируется только связанный с этим кодом процесс, а остальные процессы (исполняющиеся в собственных потоках) продолжают функционировать. Потоки потребляют дополнительные ресурсы, требуя пространства(памяти) и времени (снижая производительность среды исполнения).

Поток исполнения является довольно дорогим ресурсом и использовать его следует крайне аккуратно. Лучше всего задействовать пул потоков среды CLR, который создает и уничтожает потоки автоматически. Пул предлагает набор потоков для решения различных задач, и некоторые из этих потоков вполне справятся с решением задач вашего приложения.

## Состав потока

Каждый поток состоит из нескольких частей.

- Объект ядра потока.

- Блок окружения потока.

- Стек пользовательского режима.

- Стек режима ядра.

- Уведомления о состоянии и завершении потоков.

Компьютер с одним процессором может одновременно выполнять только что-то одно. Следовательно, операционная система должна распределять физический процессор между всеми своими потоками (логическими процессорами).

В произвольный момент времени Windows передает процессору на исполнение один поток. Этот поток исполняется в течение некоторого временного интервала, иногда называемого тактом (quantum). После завершения этого интервала контекст Windows переключается на другой поток. При этом обязательно происходит следующее:

- Значения регистров процессора исполняющегося в данный момент потока сохраняются в структуре контекста, которая располагается в ядре потока.

- Из набора имеющихся потоков выделяется тот, которому будет передано управление. Если выбранный поток принадлежит другому процессу, Windows переключает для процессора виртуальное адресное пространство. Только после этого возможно выполнение какого-либо кода или доступ к каким-либо данным.

- Значения из выбранной структуры контекста потока загружаются в регистры процессора.

При переключении контекста на другой поток производительность серьезно падает. Пока работа ведется с одним потоком, его код и данные находятся в кэше процессора, чтобы обращения процессора к оперативной памяти, замедляющие работу, происходили реже. Однако новый поток, скорее всего, исполняет совсем другой код и имеет доступ к другим данным, которых еще нет в кэше процессора. Значит, прежде чем вернуться к прежней скорости работы, процессор вынужден некоторое время обращаться к оперативной памяти, наполняя кэш.

При разработке высокопроизводительных приложений и компонентов переключения контекста нужно по возможности избегать.

Если в конце временного промежутка Windows решает продолжить исполнение уже исполняемого потока (а не переходить к другому), переключения контекста не происходит. Это значительно повышает производительность.

## CLR потоки

Заданный домен приложения может иметь многочисленные потоки, выполняющиеся в каждый конкретный момент времени. Потоки могут перемещаться между границами доменов приложений, но каждый может выполняться только внутри одного домена приложения.

Одиночный поток может быть перенесен в определенный контекст и перемещаться внутри нового контекста.

За перемещение потоков между доменами приложений и контекстами отвечает среда CLR.

Причины использования потоков:

- Улучшение времени отклика.

- Производительность.

Каждому потоку назначается уровень приоритета с нулевого (самого низкого) до 31 (самого высокого). При выборе потока, который будет передан процессору, сначала рассматриваются потоки с самым высоким приоритетом и ставятся в очередь в цикле. При обнаружении потока с приоритетом 31 он передается процессору. После завершения такта ищется следующий поток с аналогичным приоритетом, чтобы переключить на него контекст.

## Фоновые и активные потоки

В CLR все потоки делятся на активные (foreground) и фоновые (background). При завершении активных потоков в процессе CLR принудительно завершает также все запущенные на этот момент фоновые потоки. При этом завершение фоновых потоков происходит немедленно и без появления исключений.















