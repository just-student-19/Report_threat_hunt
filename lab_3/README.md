# Основы обработки данных с помощью R и Dplyr
odintsovajulia19@yandex.ru

## Цель работы

Развить практические навыки использования языка программирования R для
обработки данных 2. Закрепить знания базовых типов данных языка R 3.
Развить практические навыки использования функций обработки данных
пакета dplyr – функции select(), filter(), mutate(), arrange(),
group_by(

## Исходные данные

1.  Программное обеспечение Windows 10
2.  Rstudio Desktop
3.  Интерпретатор языка R 4.1
4.  Dplyr.
5.  Пакет данных nycflights13.

## Задание

Используя язык программирования R выполнить задания и составить отчет.

## Ход работы

-Подготовим рабочую среду

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
#install.packages('nycflights13')
library(nycflights13)
```

    Warning: пакет 'nycflights13' был собран под R версии 4.4.2

### Задание 1:Сколько встроенных в пакет nycflights13 датафреймов?

``` r
length(data(package = "nycflights13")$results[, "Item"])
```

    [1] 5

### Задание 2:Сколько строк в каждом датафрейме?

``` r
row_count = list(
  flights = nrow(flights),
  airlines = nrow(airlines),
  airports = nrow(airports),
  planes = nrow(planes),
  weather = nrow(weather))
print(row_count)
```

    $flights
    [1] 336776

    $airlines
    [1] 16

    $airports
    [1] 1458

    $planes
    [1] 3322

    $weather
    [1] 26115

### Задание 3:Сколько столбцов в каждом датафрейме?

``` r
col_count = list(
  flights = ncol(flights),
  airlines = ncol(airlines),
  airports = ncol(airports),
  planes = ncol(planes),
  weather = ncol(weather))
print(col_count)
```

    $flights
    [1] 19

    $airlines
    [1] 2

    $airports
    [1] 8

    $planes
    [1] 9

    $weather
    [1] 15

### Задание 4: Как просмотреть примерный вид датафрейма?

``` r
df_flights = flights %>% glimpse()
```

    Rows: 336,776
    Columns: 19
    $ year           <int> 2013, 2013, 2013, 2013, 2013, 2013, 2013, 2013, 2013, 2…
    $ month          <int> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1…
    $ day            <int> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1…
    $ dep_time       <int> 517, 533, 542, 544, 554, 554, 555, 557, 557, 558, 558, …
    $ sched_dep_time <int> 515, 529, 540, 545, 600, 558, 600, 600, 600, 600, 600, …
    $ dep_delay      <dbl> 2, 4, 2, -1, -6, -4, -5, -3, -3, -2, -2, -2, -2, -2, -1…
    $ arr_time       <int> 830, 850, 923, 1004, 812, 740, 913, 709, 838, 753, 849,…
    $ sched_arr_time <int> 819, 830, 850, 1022, 837, 728, 854, 723, 846, 745, 851,…
    $ arr_delay      <dbl> 11, 20, 33, -18, -25, 12, 19, -14, -8, 8, -2, -3, 7, -1…
    $ carrier        <chr> "UA", "UA", "AA", "B6", "DL", "UA", "B6", "EV", "B6", "…
    $ flight         <int> 1545, 1714, 1141, 725, 461, 1696, 507, 5708, 79, 301, 4…
    $ tailnum        <chr> "N14228", "N24211", "N619AA", "N804JB", "N668DN", "N394…
    $ origin         <chr> "EWR", "LGA", "JFK", "JFK", "LGA", "EWR", "EWR", "LGA",…
    $ dest           <chr> "IAH", "IAH", "MIA", "BQN", "ATL", "ORD", "FLL", "IAD",…
    $ air_time       <dbl> 227, 227, 160, 183, 116, 150, 158, 53, 140, 138, 149, 1…
    $ distance       <dbl> 1400, 1416, 1089, 1576, 762, 719, 1065, 229, 944, 733, …
    $ hour           <dbl> 5, 5, 5, 5, 6, 5, 6, 6, 6, 6, 6, 6, 6, 6, 6, 5, 6, 6, 6…
    $ minute         <dbl> 15, 29, 40, 45, 0, 58, 0, 0, 0, 0, 0, 0, 0, 0, 0, 59, 0…
    $ time_hour      <dttm> 2013-01-01 05:00:00, 2013-01-01 05:00:00, 2013-01-01 0…

``` r
print(df_flights)
```

    # A tibble: 336,776 × 19
        year month   day dep_time sched_dep_time dep_delay arr_time sched_arr_time
       <int> <int> <int>    <int>          <int>     <dbl>    <int>          <int>
     1  2013     1     1      517            515         2      830            819
     2  2013     1     1      533            529         4      850            830
     3  2013     1     1      542            540         2      923            850
     4  2013     1     1      544            545        -1     1004           1022
     5  2013     1     1      554            600        -6      812            837
     6  2013     1     1      554            558        -4      740            728
     7  2013     1     1      555            600        -5      913            854
     8  2013     1     1      557            600        -3      709            723
     9  2013     1     1      557            600        -3      838            846
    10  2013     1     1      558            600        -2      753            745
    # ℹ 336,766 more rows
    # ℹ 11 more variables: arr_delay <dbl>, carrier <chr>, flight <int>,
    #   tailnum <chr>, origin <chr>, dest <chr>, air_time <dbl>, distance <dbl>,
    #   hour <dbl>, minute <dbl>, time_hour <dttm>

### Задание 5: Сколько компаний-перевозчиков (carrier) учитывают эти наборы данных (представлено в наборах данных)?

``` r
carrier = 
  flights %>% 
  filter(!is.na(carrier)) %>% 
  distinct(carrier) %>% 
  nrow()
print(carrier)
```

    [1] 16

### Задание 6: Сколько рейсов принял аэропорт John F Kennedy Intl в мае?

``` r
jfk_may_flights = 
  flights %>% 
  filter(dest == "JFK", month == 5) %>% 
  nrow()
print(jfk_may_flights)
```

    [1] 0

### Задание 7: Какой самый северный аэропорт?

``` r
northernmost_airport = 
  airports %>%
  filter(lat == max(lat)) %>%
  select(faa, name, lat, lon)
print(northernmost_airport)
```

    # A tibble: 1 × 4
      faa   name                      lat   lon
      <chr> <chr>                   <dbl> <dbl>
    1 EEN   Dillant Hopkins Airport  72.3  42.9

### Задание 8: Какой аэропорт самый высокогорный (находится выше всех над уровнем моря)?

``` r
highest_airport = 
  airports %>%
  filter(alt == max(alt, na.rm = TRUE)) %>%
  mutate(alt_meters = round(alt * 0.3048)) %>%
  select(faa, name, alt, alt_meters)
print(highest_airport)
```

    # A tibble: 1 × 4
      faa   name        alt alt_meters
      <chr> <chr>     <dbl>      <dbl>
    1 TEX   Telluride  9078       2767

### Задание 9: Какие бортовые номера у самых старых самолетов?

``` r
oldest_tailnum = 
  planes %>% 
  arrange(year) %>% 
  head(1) %>% 
  select(tailnum)
print(oldest_tailnum)
```

    # A tibble: 1 × 1
      tailnum
      <chr>  
    1 N381AA 

### Задание 10: Какая средняя температура воздуха была в сентябре в аэропорту John F Kennedy Intl (в градусах Цельсия)?

``` r
jfk_sept_weather_f = 
  weather %>%
  filter(origin == "JFK", month == 9) %>%
  summarise(average_temp = mean(temp, na.rm = TRUE))
jfk_sept_weather_c =(jfk_sept_weather_f - 32) * 5 / 9
print(jfk_sept_weather_c)
```

      average_temp
    1     19.38764

### Задание 11: Самолеты какой авиакомпании совершили больше всего вылетов в июне?

``` r
top_airline_june = 
  flights %>%
  filter(month == 6) %>%
  group_by(carrier) %>%
  summarise(num_flights = n()) %>%
  arrange(desc(num_flights)) %>%
  slice(1)
print(top_airline_june)
```

    # A tibble: 1 × 2
      carrier num_flights
      <chr>         <int>
    1 UA             4975

### Задание 12: Самолеты какой авиакомпании задерживались чаще других в 2013 году?

``` r
delayed_airline_depart = 
  flights %>%
  filter(dep_delay > 0) %>%  
  group_by(carrier) %>%
  summarise(num_delays = n()) %>%
  arrange(desc(num_delays)) %>%
  slice(1) %>%
  mutate(delay_type = "Departure") %>% 
  select(carrier, delay_type, num_delays)

delayed_airline_arrive = 
  flights %>%
  filter(arr_delay > 0) %>%  
  group_by(carrier) %>%
  summarise(num_delays = n()) %>%
  arrange(desc(num_delays)) %>%
  slice(1) %>%
  mutate(delay_type = "Arrival") %>%
  select(carrier, delay_type, num_delays)

delayed_airlines = bind_rows(delayed_airline_depart, delayed_airline_arrive)

print(delayed_airlines)
```

    # A tibble: 2 × 3
      carrier delay_type num_delays
      <chr>   <chr>           <int>
    1 UA      Departure       27261
    2 EV      Arrival         24484

## Оценка результата

С использованием инструментов dplyr и RStudio были проделаны задания по
основам обработки данных.

## Вывод

Провели работу с dplyr, пакетом nycflights13, ознакомились с
технологиями подготовки и анализа данных.
