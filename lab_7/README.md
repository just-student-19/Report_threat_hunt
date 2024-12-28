# Анализ данных сетевого трафика при помощи библиотеки Arrow
odintsovajulia19@yandex.ru

## Цель работы

1.  Изучить возможности технологии Apache Arrow для обработки и анализ
    больших данных
2.  Получить навыки применения Arrow совместно с языком программирования
    R
3.  Получить навыки анализа метаинфомации о сетевом трафике

## Исходные данные

1.  Программное обеспечение Windows 10

2.  Rstudio Desktop

3.  Интерпретатор языка R 4.1

4.  Apache Arrow

5.  tm_data.pqt

## Задание

Используя язык программирования R, библиотеку arrow и IDE Rstudio
Destop, выполнить задания и составить отчет.

## Ход работы

### Подготовка рабочей среды

-Подготовим рабочую среду

``` r
library(arrow)
```

    Warning: пакет 'arrow' был собран под R версии 4.4.2


    Присоединяю пакет: 'arrow'

    Следующий объект скрыт от 'package:utils':

        timestamp

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
data = arrow::open_dataset(sources = "tm_data.pqt", format = "parquet")
```

### Задание 1: Надите утечку данных из Вашей сети

Важнейшие документы с результатами нашей исследовательской деятельности
в области создания вакцин скачиваются в виде больших заархивированных
дампов. Один из хостов в нашей сети используется для пересылки этой
информации – он пересылает гораздо больше информации на внешние ресурсы
в Интернете, чем остальные компьютеры нашей сети. Определите его
IP-адрес.

``` r
leak_1 = 
  data %>%
     group_by(src) %>%
     summarise(sum = sum(bytes, na.rm = TRUE), .groups = 'drop') %>%
     arrange(desc(sum))

 leak_1_df = as.data.frame(leak_1)
 result_1 = 
   slice( leak_1_df, 1) %>%
     select(src)
print(result_1)
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
work_hours = 
  data %>%
    select(timestamp, src, dst, bytes) %>%
    mutate(hour = hour(as_datetime(timestamp / 1000))) %>%
    filter(hour >= 0 & hour < 24) %>%
    group_by(hour) %>%
    summarise(work_hours = n(), .groups = 'drop') %>%
    arrange(desc(work_hours))

work_hours_df = work_hours %>% collect()
print(work_hours_df)
```

    # A tibble: 24 × 2
        hour work_hours
       <int>      <int>
     1    22   12237573
     2    23   12226575
     3    18   12226457
     4    21   12224721
     5    20   12220671
     6    19   12219218
     7    16   12217746
     8    17   12213523
     9     8     537639
    10     9     495908
    # ℹ 14 more rows

-   Понимаем, что для поиска будет использоваться временной интервал 0 -
    15

``` r
leak_2 = 
  data %>%
  mutate(hour = hour(as_datetime(timestamp / 1000))) %>%
  filter(!str_detect(src, "^13\\.37\\.84\\.125")) %>%
  filter(hour >= 1 & hour <= 15) %>%
  group_by(src) %>%
  summarise(sum = sum(bytes), .groups = 'drop') %>%
  arrange(desc(sum)) %>%
  head(1) %>%
  select(src)

leak_2_df = leak_2 %>% collect()
print(leak_2_df)
```

    # A tibble: 1 × 1
      src        
      <chr>      
    1 12.55.77.96

### Задание 3: Надите утечку данных 3

Еще один нарушитель собирает содержимое электронной почты и отправляет в
Интернет используя порт, который обычно используется для другого типа
трафика. Атакующий пересылает большое количество информации используя
этот порт, которое нехарактерно для других хостов, использующих этот
номер порта. Определите IP этой системы. Известно, что ее IP адрес
отличается от нарушителей из предыдущих задач.

``` r
unsafe_ports = 
  data %>%
  filter(!str_detect(src, "^13\\.37\\.84\\.125")) %>%
  filter(!str_detect(src, "^12\\.55\\.77\\.96")) %>%
  select(src, bytes, port)

unsafe_ports_agr = 
  unsafe_ports %>%
  group_by(port) %>%
  summarise(
    mean = mean(bytes, na.rm = TRUE),
    max = max(bytes, na.rm = TRUE),
    sum = sum(bytes, na.rm = TRUE),
    .groups = 'drop'
  ) %>%
  mutate(diff = max - mean) %>%
  filter(diff != 0) %>%
  arrange(desc(diff)) %>%
  head(1)

unsafe_ports_df = unsafe_ports_agr %>% collect()
print(unsafe_ports_df)
```

    # A tibble: 1 × 5
       port   mean    max          sum    diff
      <int>  <dbl>  <int>      <int64>   <dbl>
    1   119 31613. 209446 110302445799 177833.

-   Пришли к выводу, что порт 119 наиболее вероятно используется
    нарушителем. Посмотрим информацию о нем

``` r
leak_3 = 
  unsafe_ports %>%
  filter(port == 119) %>%
  group_by(src) %>%
  summarise(mean = mean(bytes, na.rm = TRUE), .groups = 'drop') %>%
  arrange(desc(mean)) %>%
  head(1) %>%
  select(src)

leak_3_df = leak_3 %>% collect()
print(leak_3_df)
```

    # A tibble: 1 × 1
      src        
      <chr>      
    1 18.68.32.32

## Оценка результата

С использованием инструментов arrow были проделаны задания по
исследованию сетевого трафика.

## Вывод

Провели работу с arrow, проанализировали информацию о сетевом трафике.
