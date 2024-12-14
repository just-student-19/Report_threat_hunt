# Анализ данных сетевого трафика при помощи библиотеки Arrow
odintsovajulia19@yandex.ru

## Цель работы

1.  Изучить возможности технологии Apache Arrow для обработки и анализ
    больших данных
2.  Получить навыки применения Arrow совместно с языком программирования
    R
3.  Получить навыки анализа метаинфомации о сетевом трафике
4.  Получить навыки применения облачных технологий хранения, подготовки
    и анализа данных: Yandex Object Storage, Rstudio Server.

## Исходные данные

1.  Программное обеспечение Windows 10
2.  Rstudio Server(использовалась учетка user35)
3.  Интерпретатор языка R 4.1
4.  Apache Arrow

## Задание

Используя язык программирования R, библиотеку arrow и облачную IDE
Rstudio Server, развернутую в Yandex Cloud, выполнить задания и
составить отчет.

## Ход работы

### Задание 1: Надите утечку данных из Вашей сети

Важнейшие документы с результатами нашей исследовательской деятельности
в области создания вакцин скачиваются в виде больших заархивированных
дампов. Один из хостов в нашей сети используется для пересылки этой
информации – он пересылает гораздо больше информации на внешние ресурсы
в Интернете, чем остальные компьютеры нашей сети. Определите его
IP-адрес.

-Подготовим рабочую среду

``` r
library(arrow)
```

    Some features are not enabled in this build of Arrow. Run `arrow_info()` for more information.


    Attaching package: 'arrow'

    The following object is masked from 'package:utils':

        timestamp

``` r
library(dplyr)
```


    Attaching package: 'dplyr'

    The following objects are masked from 'package:stats':

        filter, lag

    The following objects are masked from 'package:base':

        intersect, setdiff, setequal, union

``` r
library(stringr)
library(knitr)
download.file("https://storage.yandexcloud.net/arrow-datasets/tm_data.pqt",destfile = "tm_data.pqt")
df <- arrow::open_dataset(sources = "tm_data.pqt", format = "parquet")
```

``` r
task1 <- df %>%
     filter(
         str_detect(src, "^12\\.|^13\\.|^14\\.") &
             !str_detect(dst, "^12\\.|^13\\.|^14\\.")
     ) %>%
     group_by(src) %>%
     summarise(sum = sum(bytes, na.rm = TRUE), .groups = 'drop') %>%
     arrange(desc(sum))
# Преобразование в датафрейм для использования slice
 task1_df <- as.data.frame(task1)
 result <- slice(task1_df, 1) %>%
     select(src)
 
# Выбор первой строки
 result %>%
     knitr::kable()
```

<table>
<thead>
<tr class="header">
<th style="text-align: left;">src</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">13.37.84.125</td>
</tr>
</tbody>
</table>

### Задание 2: Надите утечку данных 2

Другой атакующий установил автоматическую задачу в системном
планировщике cron для экспорта содержимого внутренней wiki системы. Эта
система генерирует большое количество трафика в нерабочие часы, больше
чем остальные хосты. Определите IP этой системы. Известно, что ее IP
адрес отличается от нарушителя из предыдущей задачи.

``` r
task2_1 <- df %>%
    select(timestamp, src, dst, bytes) %>%
    mutate(
        trafic = str_detect(src, "^((12|13|14)\\.)") & !str_detect(dst, "^((12|13|14)\\.)"),
        time = hour(as_datetime(timestamp / 1000))
    ) %>%
    filter(trafic == TRUE, time >= 0 & time < 24) %>%
    group_by(time) %>%
    summarise(trafictime = n(), .groups = 'drop') %>%
    arrange(desc(trafictime))
# Используем collect() для получения результата в виде датафрейма
task2_1_df <- task2_1 %>% collect()
# Отображение результатов
task2_1_df %>%
    knitr::kable()
```

<table>
<thead>
<tr class="header">
<th style="text-align: right;">time</th>
<th style="text-align: right;">trafictime</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: right;">16</td>
<td style="text-align: right;">4490576</td>
</tr>
<tr class="even">
<td style="text-align: right;">22</td>
<td style="text-align: right;">4489703</td>
</tr>
<tr class="odd">
<td style="text-align: right;">18</td>
<td style="text-align: right;">4489386</td>
</tr>
<tr class="even">
<td style="text-align: right;">23</td>
<td style="text-align: right;">4488093</td>
</tr>
<tr class="odd">
<td style="text-align: right;">19</td>
<td style="text-align: right;">4487345</td>
</tr>
<tr class="even">
<td style="text-align: right;">21</td>
<td style="text-align: right;">4487109</td>
</tr>
<tr class="odd">
<td style="text-align: right;">17</td>
<td style="text-align: right;">4483578</td>
</tr>
<tr class="even">
<td style="text-align: right;">20</td>
<td style="text-align: right;">4482712</td>
</tr>
<tr class="odd">
<td style="text-align: right;">13</td>
<td style="text-align: right;">169617</td>
</tr>
<tr class="even">
<td style="text-align: right;">7</td>
<td style="text-align: right;">169241</td>
</tr>
<tr class="odd">
<td style="text-align: right;">0</td>
<td style="text-align: right;">169068</td>
</tr>
<tr class="even">
<td style="text-align: right;">3</td>
<td style="text-align: right;">169050</td>
</tr>
<tr class="odd">
<td style="text-align: right;">14</td>
<td style="text-align: right;">169028</td>
</tr>
<tr class="even">
<td style="text-align: right;">6</td>
<td style="text-align: right;">169015</td>
</tr>
<tr class="odd">
<td style="text-align: right;">12</td>
<td style="text-align: right;">168892</td>
</tr>
<tr class="even">
<td style="text-align: right;">10</td>
<td style="text-align: right;">168750</td>
</tr>
<tr class="odd">
<td style="text-align: right;">2</td>
<td style="text-align: right;">168711</td>
</tr>
<tr class="even">
<td style="text-align: right;">11</td>
<td style="text-align: right;">168684</td>
</tr>
<tr class="odd">
<td style="text-align: right;">1</td>
<td style="text-align: right;">168539</td>
</tr>
<tr class="even">
<td style="text-align: right;">4</td>
<td style="text-align: right;">168422</td>
</tr>
<tr class="odd">
<td style="text-align: right;">15</td>
<td style="text-align: right;">168355</td>
</tr>
<tr class="even">
<td style="text-align: right;">9</td>
<td style="text-align: right;">168283</td>
</tr>
<tr class="odd">
<td style="text-align: right;">5</td>
<td style="text-align: right;">168283</td>
</tr>
<tr class="even">
<td style="text-align: right;">8</td>
<td style="text-align: right;">168205</td>
</tr>
</tbody>
</table>

-   Понимаем, что для поиска будет использоваться временной интервал 0 -
    15

``` r
task2_2 <- df %>%
  mutate(time = hour(as_datetime(timestamp / 1000))) %>%
  filter(!str_detect(src, "^13\\.37\\.84\\.125")) %>%
  filter(str_detect(src, "^12\\.") | str_detect(src, "^13\\.") | str_detect(src, "^14\\.")) %>%
  filter(!str_detect(dst, "^12\\.") & !str_detect(dst, "^13\\.") & !str_detect(dst, "^14\\.")) %>%
  filter(time >= 1 & time <= 15) %>%
  group_by(src) %>%
  summarise(sum = sum(bytes), .groups = 'drop') %>%
  arrange(desc(sum)) %>%
  head(1) %>%
  select(src)

# Используем collect() для получения результата в виде датафрейма
task2_2_df <- task2_2 %>% collect()
# Отображение результатов
task2_2_df %>%
  knitr::kable()
```

<table>
<thead>
<tr class="header">
<th style="text-align: left;">src</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">12.55.77.96</td>
</tr>
</tbody>
</table>

### Задание 3: Надите утечку данных 3

Еще один нарушитель собирает содержимое электронной почты и отправляет в
Интернет используя порт, который обычно используется для другого типа
трафика. Атакующий пересылает большое количество информации используя
этот порт, которое нехарактерно для других хостов, использующих этот
номер порта. Определите IP этой системы. Известно, что ее IP адрес
отличается от нарушителей из предыдущих задач.

``` r
task3_1 <- df %>%
  filter(!str_detect(src, "^13\\.37\\.84\\.125")) %>%
  filter(!str_detect(src, "^12\\.55\\.77\\.96")) %>%
  filter(str_detect(src, "^12\\.") | str_detect(src, "^13\\.") | str_detect(src, "^14\\.")) %>%
  filter(!str_detect(dst, "^12\\.") & !str_detect(dst, "^13\\.") & !str_detect(dst, "^14\\.")) %>%
  select(src, bytes, port)

# Группировка и агрегация данных
task3_1_summary <- task3_1 %>%
  group_by(port) %>%
  summarise(
    mean = mean(bytes, na.rm = TRUE),
    max = max(bytes, na.rm = TRUE),
    sum = sum(bytes, na.rm = TRUE),
    .groups = 'drop'
  ) %>%
  mutate(Raz = max - mean) %>%
  filter(Raz != 0) %>%
  arrange(desc(Raz)) %>%
  head(1)

# Используем collect() для получения результата в виде датафрейма
task3_1_result <- task3_1_summary %>% collect()

# Отображение результатов
task3_1_result %>%
  knitr::kable()
```

<table>
<thead>
<tr class="header">
<th style="text-align: right;">port</th>
<th style="text-align: right;">mean</th>
<th style="text-align: right;">max</th>
<th style="text-align: right;">sum</th>
<th style="text-align: right;">Raz</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: right;">37</td>
<td style="text-align: right;">35089.99</td>
<td style="text-align: right;">209402</td>
<td style="text-align: right;">32136394510</td>
<td style="text-align: right;">174312</td>
</tr>
</tbody>
</table>

-   Пришли к выводу, что порт 37 наиболее вероятно используется
    нарушителем. Посмотрим информацию о нем

``` r
task3_2 <- task3_1 %>%
  filter(port == 37) %>%
  group_by(src) %>%
  summarise(mean = mean(bytes, na.rm = TRUE), .groups = 'drop') %>%
  arrange(desc(mean)) %>%
  head(1) %>%
  select(src)

# Используем collect() для получения результата в виде датафрейма
task3_2_result <- task3_2 %>% collect()
# Отображение результатов
task3_2_result %>%
  knitr::kable()
```

<table>
<thead>
<tr class="header">
<th style="text-align: left;">src</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">14.31.107.42</td>
</tr>
</tbody>
</table>

## Оценка результата

С использованием инструментов arrow были проведены задания по
исследованию сетевого трафика.

## Вывод

Провели работу с arrow, ознакомились с применением облачных технологий
для хранения, подготовки и анализа данных, а также проанализировали
метаинформацию о сетевом трафике.
