## 200.1 Измерение производительности и решение проблем связанных с ней

Студент должен уметь измерять производительность железа, пропускную способность сети, определять и исправлять возникающие проблемы.

**Изучаем** :

- метрики ЦПУ;
- метрики ОЗУ;
- метрики HDD;

Вне зависимости от используемых инструментов снятия метрик, для определения производительности CPU используются следующие характеристики:

- **user** : время, затраченное на исполнения кода на уровне пользователя (чаще всего код софта, не использующий железо);
- **system** : время, затраченное на исполнение кода на уровне ядра (чаще всего это ОС – операции ввода вывода, управление процессами и тд);
- **nice** : время, затраченное на выполнение операций с измененным приоритетом;
- **idle** : время простаивания CPU;
- **wait** : время ожидания ввода-вывода (чаще всего ожидание ответа от дисковой подсистемы);
- **hardware_interrupts** : время затраченное на обработку аппаратных прерываний (события от железа);
- **software_interrupts** : время затраченное на обработку программных прерываний (события от программного кода);
- **stolen** : время ожидания ответа от гипервизора (в случае виртуальной машины).
- **load average** : показывает 'занятость' процессора за последние 1, 5 и 15 минут и позволяет определить вектор нагрузки - увеличивается/уменьшается

Для определения базовых параметров производительности можно использовать утилиты **vmstat** , **top** и модный **htop**. Для тестовой загрузки процессора мы использовали пакет **stress**.

#### Примеры
Команда | Описание
--- | ---
**sar** *1* | Загрузка процессора
**iostat** *-c 1*| Загрузка процессора
**sar** *-P ALL* | Загрузка процессора по ядрам
**sar** *-q 1* | Load Average
**w, uptime, tload** | Load Average
**ps** *-eo pid,%cpu,cputime,%mem,rss,vsz,comm --sort -%cpu \| head -5 \| column -t* | Топ-5 процессов по загрузке CPU

------------------------------

Вне зависимости от используемых инструментов снятия метрик, для определения производительности оперативной памяти используются следующие характеристики:

- **total** : всего ОЗУ;
- **free** : реально свободно ОЗУ (т.е. количество никак не задействованной памяти, Linux стремится уменьшить ее до минимума);
- **used** : используется ОЗУ;
- **shared** : память разделяемая процессами (чаще всего используется для взаимодействия между процессами во избежание использования системных вызовов ядра);
- **cached** : кэшированная ОЗУ (например, набор данных, к которым часто обращается программа, может быть помещен в кэш ОЗУ с жесткого диска, для более быстрого доступа к ним);
- **buffered** : буферизированная ОЗУ (например, промежуточное хранилище данных перед обработкой, или перемещением их на диск). Часто можно увидеть buffered/cached как единое целое, логически показывающее ту область памяти, которую можно освободить при необходимости.
- **available** : количество памяти которую можно использовать без необходимости обращаться к swap (т.е. память, которая будет свободна если выкинуть все из кэшей и т.д.);
- **active** : память, активно используемая процессом;
- **inactive** : память, которая была выделена под процесс, но в данный момент им больше не используется;
- **swapped** : в свопе (на жестком диске – в разделе или в файле);

Для раздела (или файла) подкачки могут быть использованы метрики:

- **swapped_in** : количество блоков, считанных за секунду с диска;
- **swapped_out** : количество блоков, записанных  за секунду на диск;

Здесь и далее: размер блока указывается при форматировании файловой системы, чаще всего его размер равен 4Кб.

\* _Мне кажется, что swapped_in/out всё-таки измеряется в **страницах** памяти, но размер страницы тоже равен 4Кб. Размер же блока(сектора) зависит от самого устройства хранения и может быть 512/4096 байт, и именно этот блок является единицей записи на диск. А если говорить о блоке файловой системы (кластере) то он вообще может принимать различные значения и может задаваться при форматировании, в зависимости от размера раздела и его назначения. //eugenenuke_

[https://ru.wikipedia.org/wiki/Сектор_диска](https://ru.wikipedia.org/wiki/%D0%A1%D0%B5%D0%BA%D1%82%D0%BE%D1%80_%D0%B4%D0%B8%D1%81%D0%BA%D0%B0)

[https://toster.ru/q/19776](https://toster.ru/q/19776)

Удобным инструментом для мониторинга состояния ОЗУ является **free**. Например:

_free -m_

#### Примеры
Команда | Описание
--- | ---
free -m | Информация о доступной/занятой памяти
vmstat 1 | Информация о доступной/занятой памяти
vmstat -s | Статистика по памяти
sar -r 1 | Информация по использованию памяти
sar -S 1 | Информация по использованию swap
swapon -s | Информация по использованию swap
ps -eo pid,%cpu,cputime,%mem,rss,vsz,comm --sort -%mem \| head -5 \| column -t | Топ-5 процессов по использованию памяти

------------------------------

Вне зависимости от используемых инструментов снятия метрик, для определения производительности жестких дисков (если конкретно – блочных устройств) используются следующие характеристики:

- **blocks_in** : количество считанных блоков в единицу времени;
- **blocks_out** : количество записанных блоков в единицу времени;
- **total_read/write** : скорость чтения/записи между процессами и подсистемой ввода/вывода;
- **actual_read/write** : скорость чтения/записи между подсистемой ввода/вывода и самими дисками;
- **tid** : **t**hread**i**d (идентификатор потока процесса ввода/вывода, аналогичный pid для вычислительных процессов);
- **prio** : приоритет (аналогично nice для процессов). Бывает idle (минимальный), be/0-7 (**b**est**e**ffort, чем меньше число тем выше приоритет) и rt/0-7 (**r**eal**t**ime, максимальный, чем меньше число тем выше приоритет)
- **swapin** : используемая часть swap;
- **tps** : количество запросов к диску в секунду;
- **rrqm/s** : количество запросов на чтение в секунду;
- **wrqm/s** : количество запросов на запись в секунду;
- **r/s** : количество операций чтения в секунду;
- **w/s** : количество операций записи в секунду;
- **rkB/s** : скорость чтения в килобайт/сек;
- **wkB/s** : скорость записи в килобайт/сек;
- **avgrq-sz** : средний размер запроса к диску в секторах;
- **avgqu-sz** : средняя очередь запросов к диску в секторах;
- **await** : среднее время обслуживания запроса (включая нахождение в очереди на обработку) в миллисекундах;
- **r_await** : среднее время обслуживания запроса на чтение (включая нахождение в очереди на обработку) в миллисекундах;
- **w_await** : среднее время обслуживания запроса на запись (включая нахождение в очереди на обработку) в миллисекундах;
- **svctm** : среднее время обслуживания запроса в миллисекундах;
- **util** : условно говоря – насколько был занят диск (по аналогии с утилизацией процессора);

Удобным инструментом для мониторинга состояния жестких дисков является **iotop** (ставится отдельным пакетом), **iostat** и **sar** (входят в состав пакета **sysstat** ). Например:

#### Примеры
Команда | Описание
--- | ---
**iostat** *-x 1* | Расширенная статистика по вводу/выводу
**sar** *-b 1* | Статистика по вводу/выводу блочных устройств (суммарно)
**sar** *-dp 1* | Статистика по вводу/выводу блочных устройств (по устройству)
**iostat** *-d 1* | Статистика по вводу/выводу блочных устройств (по устройству)
**vmstat** *-d 1* | Статистика по вводу/выводу блочных устройств (по устройству)
**iostat** *-dp 1* | Статистика по вводу/выводу блочных устройств (по разделам)
**vmstat** *-D* | Суммарная статистика по вводу/выводу
