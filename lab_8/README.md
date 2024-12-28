# Анализ данных сетевого трафика с использованием аналитической
in-memory СУБД DuckDB
odintsovajulia19@yandex.ru


## Цель работы

1.  Изучить возможности СУБД DuckDB для обработки и анализ больших
    данных
2.  Получить навыки применения DuckDB совместно с языком
    программирования R
3.  Получить навыки анализа метаинфомации о сетевом трафике

## Исходные данные

1.  Программное обеспечение Windows 10
2.  Данные tm_data.pqt
3.  Rstudio Desktop
4.  Интерпретатор языка R 4.1
5.  DuckDB и Dplyr.

## Задание

Используя язык программирования R, OLAP СУБД
DuckDB библиотеку duckdb и IDE Rstudio
Desktop, выполнить задания
и составить отчет.


## Ход работы

### Подготовим рабочую среду

``` r
library(duckdb)
```

    Warning: пакет 'duckdb' был собран под R версии 4.4.2

    Загрузка требуемого пакета: DBI

    Warning: пакет 'DBI' был собран под R версии 4.4.2

``` r
library(dplyr)
```

    Warning: пакет 'dplyr' был собран под R версии 4.4.2


    Присоединяю пакет: 'dplyr'

    Следующие объекты скрыты от 'package:stats':

        filter, lag

    Следующие объекты скрыты от 'package:base':

        intersect, setdiff, setequal, union

``` r
download.file("https://storage.yandexcloud.net/arrow-datasets/tm_data.pqt",destfile = "tm_data.pqt")
con <- dbConnect(duckdb())
dbExecute(con,"CREATE TABLE new_tbl as SELECT * FROM read_parquet('tm_data.pqt')")
```

    [1] 105747730

### Задание 1: Надите утечку данных из Вашей сети

Важнейшие документы с результатами нашей исследовательской деятельности
в области создания вакцин скачиваются в виде больших заархивированных
дампов. Один из хостов в нашей сети используется для пересылки этой
информации – он пересылает гораздо больше информации на внешние ресурсы
в Интернете, чем остальные компьютеры нашей сети. Определите его
IP-адрес.

``` r
leak_1 = dbGetQuery(con,
"SELECT src FROM new_tbl
GROUP BY src
order by sum(bytes) desc
limit 1") 
print(leak_1)
```

               src
    1 13.37.84.125

### Задание 2: Надите утечку данных 2

Другой атакующий установил автоматическую задачу в системном
планировщике cron для экспорта содержимого внутренней wiki системы. Эта
система генерирует большое количество трафика в нерабочие часы, больше
чем остальные хосты. Определите IP этой системы. Известно, что ее IP
адрес отличается от нарушителя из предыдущей задачи.

``` r
work_hours = dbGetQuery(con,
"SELECT hour, COUNT(*) AS work_hours
FROM (
    SELECT 
        timestamp,
        src,
        dst,
        bytes,
        EXTRACT(HOUR FROM epoch_ms(CAST(timestamp AS BIGINT))) AS hour
    FROM new_tbl
) sub
WHERE hour BETWEEN 0 AND 24
GROUP BY hour
ORDER BY work_hours DESC;") 
print(work_hours)
```

       hour work_hours
    1    22   12237573
    2    23   12226575
    3    18   12226457
    4    21   12224721
    5    20   12220671
    6    19   12219218
    7    16   12217746
    8    17   12213523
    9     8     537639
    10    9     495908
    11   10     495765
    12   13     495660
    13   14     495598
    14   12     495317
    15    0     495178
    16    2     495122
    17    7     494972
    18    1     494821
    19    4     494691
    20    6     494401
    21   11     494401
    22    3     494361
    23   15     493787
    24    5     493625

-   Понимаем, что для поиска будет использоваться временной интервал 0 -
    15

``` r
leak_2 = dbGetQuery(con,
"SELECT src
FROM (
    SELECT src, SUM(bytes) AS trafic
    FROM (
        SELECT *,
            EXTRACT(HOUR FROM epoch_ms(CAST(timestamp AS BIGINT))) AS hour
        FROM new_tbl
    ) sub
    WHERE src <> '13.37.84.125' AND hour BETWEEN 1 AND 15
    GROUP BY src
) grp
ORDER BY trafic DESC
LIMIT 1;") 
print(leak_2)
```

              src
    1 12.55.77.96

### Задание 3: Надите утечку данных 3

Еще один нарушитель собирает содержимое электронной почты и отправляет в
Интернет используя порт, который обычно используется для другого типа
трафика. Атакующий пересылает большое количество информации используя
этот порт, которое нехарактерно для других хостов, использующих этот
номер порта. Определите IP этой системы. Известно, что ее IP адрес
отличается от нарушителей из предыдущих задач.

``` r
exclude_hosts = dbExecute(con,
"CREATE TEMPORARY TABLE exclude_hosts AS
SELECT src, bytes, port
FROM new_tbl
WHERE src <> '13.37.84.125'
    AND src <> '12.55.77.96';")

unsafe_ports = dbGetQuery(con,
"SELECT port, AVG(bytes) AS mean_bytes, 
MAX(bytes) AS max_bytes, 
SUM(bytes) AS sum_bytes, 
MAX(bytes) - AVG(bytes) AS diff
FROM exclude_hosts
GROUP BY port
HAVING diff != 0
ORDER BY diff DESC
LIMIT 1;") 
print(unsafe_ports)
```

      port mean_bytes max_bytes    sum_bytes     diff
    1  119   31612.51    209446 110302445799 177833.5

-   Пришли к выводу, что порт 119 наиболее вероятно используется
    нарушителем. Посмотрим информацию о нем

``` r
leak_3 = dbGetQuery(con,"SELECT src
FROM (
    SELECT src, AVG(bytes) AS mean
    FROM exclude_hosts
    WHERE port = 119
    GROUP BY src
) AS leak_3
ORDER BY mean DESC
LIMIT 1;") 
print(leak_3)
```

              src
    1 18.68.32.32

## Оценка результата

С использованием инструментов DuckDB и RStudio Server были проведены
задания по исследованию сетевого трафика.

## Вывод

Провели работу с DuckDB, ознакомились с применением облачных технологий
для хранения, подготовки и анализа данных, а также проанализировали
метаинформацию о сетевом трафике.
