# Исследование информации о состоянии беспроводных сетей
odintsovajulia19@yandex.ru

## Цель работы

1.  Получить знания о методах исследования радиоэлектронной обстановки.
2.  Составить представление о механизмах работы Wi-Fi сетей на канальном
    и сетевом уровне модели OSI.
3.  Зекрепить практические навыки использования языка программирования R
    для обработки данных
4.  Закрепить знания основных функций обработки данных экосистемы языка
    R

## Исходные данные

1.  Программное обеспечение Windows 10
2.  Rstudio Desktop
3.  Интерпретатор языка R 4.1
4.  Dplyr.
5.  Данные p2_wifi_data.csv.

## Задание

Используя программный пакет dplyr языка программирования R провести
анализ журналов и ответить на вопросы.

## Ход работы

### Подготовка рабочей среды

``` r
library(tidyverse)
```

    Warning: пакет 'tidyverse' был собран под R версии 4.4.2

    Warning: пакет 'ggplot2' был собран под R версии 4.4.2

    Warning: пакет 'tidyr' был собран под R версии 4.4.2

    Warning: пакет 'readr' был собран под R версии 4.4.2

    Warning: пакет 'purrr' был собран под R версии 4.4.2

    Warning: пакет 'dplyr' был собран под R версии 4.4.2

    Warning: пакет 'forcats' был собран под R версии 4.4.2

    Warning: пакет 'lubridate' был собран под R версии 4.4.2

    ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ✔ dplyr     1.1.4     ✔ readr     2.1.5
    ✔ forcats   1.0.0     ✔ stringr   1.5.1
    ✔ ggplot2   3.5.1     ✔ tibble    3.2.1
    ✔ lubridate 1.9.4     ✔ tidyr     1.3.1
    ✔ purrr     1.0.2     
    ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ✖ dplyr::filter() masks stats::filter()
    ✖ dplyr::lag()    masks stats::lag()
    ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

-   Импортируем данные и приведем датасеты в вид “аккуратных данных”

``` r
data1 = read.csv("P2_wifi_data.csv", nrows = 167)

data1 = data1 %>%
  mutate(across(c(BSSID, Privacy, Cipher, Authentication, LAN.IP, ESSID), 
                 \(x) trimws(x))) %>%
  mutate(across(c(BSSID, Privacy, Cipher, Authentication, LAN.IP, ESSID), 
                 \(x) na_if(x, "")))

data1$First.time.seen = as.POSIXct(data1$First.time.seen, format = "%Y-%m-%d %H:%M:%S")
data1$Last.time.seen = as.POSIXct(data1$Last.time.seen, format = "%Y-%m-%d %H:%M:%S")
```

``` r
data2 = read.csv("P2_wifi_data.csv", skip = 170)

data2 = data2 %>%
  mutate(across(c(Station.MAC, BSSID, Probed.ESSIDs), 
                 \(x) trimws(x))) %>%
  mutate(across(c(Station.MAC, BSSID, Probed.ESSIDs), 
                 \(x) na_if(x, "")))

data2$First.time.seen = as.POSIXct(data2$First.time.seen, format = "%Y-%m-%d %H:%M:%S")
data2$Last.time.seen = as.POSIXct(data2$Last.time.seen, format = "%Y-%m-%d %H:%M:%S")
```

``` r
glimpse(data1)
```

    Rows: 167
    Columns: 15
    $ BSSID           <chr> "BE:F1:71:D5:17:8B", "6E:C7:EC:16:DA:1A", "9A:75:A8:B9…
    $ First.time.seen <dttm> 2023-07-28 09:13:03, 2023-07-28 09:13:03, 2023-07-28 …
    $ Last.time.seen  <dttm> 2023-07-28 11:50:50, 2023-07-28 11:55:12, 2023-07-28 …
    $ channel         <int> 1, 1, 1, 7, 6, 6, 11, 11, 11, 1, 6, 14, 11, 11, 6, 6, …
    $ Speed           <int> 195, 130, 360, 360, 130, 130, 195, 130, 130, 195, 180,…
    $ Privacy         <chr> "WPA2", "WPA2", "WPA2", "WPA2", "WPA2", "OPN", "WPA2",…
    $ Cipher          <chr> "CCMP", "CCMP", "CCMP", "CCMP", "CCMP", NA, "CCMP", "C…
    $ Authentication  <chr> "PSK", "PSK", "PSK", "PSK", "PSK", NA, "PSK", "PSK", "…
    $ Power           <int> -30, -30, -68, -37, -57, -63, -27, -38, -38, -66, -42,…
    $ X..beacons      <int> 846, 750, 694, 510, 647, 251, 1647, 1251, 704, 617, 13…
    $ X..IV           <int> 504, 116, 26, 21, 6, 3430, 80, 11, 0, 0, 86, 0, 0, 0, …
    $ LAN.IP          <chr> "0.  0.  0.  0", "0.  0.  0.  0", "0.  0.  0.  0", "0.…
    $ ID.length       <int> 12, 4, 2, 14, 25, 13, 12, 13, 24, 12, 10, 0, 24, 24, 1…
    $ ESSID           <chr> "C322U13 3965", "Cnet", "KC", "POCO X5 Pro 5G", NA, "M…
    $ Key             <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…

``` r
glimpse(data2)
```

    Rows: 12,269
    Columns: 7
    $ Station.MAC     <chr> "CA:66:3B:8F:56:DD", "96:35:2D:3D:85:E6", "5C:3A:45:9E…
    $ First.time.seen <dttm> 2023-07-28 09:13:03, 2023-07-28 09:13:03, 2023-07-28 …
    $ Last.time.seen  <dttm> 2023-07-28 10:59:44, 2023-07-28 09:13:03, 2023-07-28 …
    $ Power           <chr> " -33", " -65", " -39", " -61", " -53", " -43", " -31"…
    $ X..packets      <chr> "      858", "        4", "      432", "      958", " …
    $ BSSID           <chr> "BE:F1:71:D5:17:8B", "(not associated)", "BE:F1:71:D6:…
    $ Probed.ESSIDs   <chr> "C322U13 3965", "IT2 Wireless", "C322U21 0566", "C322U…

## Точки доступа

### Задание 1: Определить небезопасные точки доступа (без шифрования – OPN)

``` r
unsafe_access_points = 
  data1 %>% 
  filter(grepl("OPN", Privacy)) %>% 
  select(BSSID, Privacy)  

print(unsafe_access_points) 
```

                   BSSID Privacy
    1  E8:28:C1:DC:B2:52     OPN
    2  E8:28:C1:DC:B2:50     OPN
    3  E8:28:C1:DC:B2:51     OPN
    4  E8:28:C1:DC:FF:F2     OPN
    5  00:25:00:FF:94:73     OPN
    6  E8:28:C1:DD:04:52     OPN
    7  E8:28:C1:DE:74:31     OPN
    8  E8:28:C1:DE:74:32     OPN
    9  E8:28:C1:DC:C8:32     OPN
    10 E8:28:C1:DD:04:50     OPN
    11 E8:28:C1:DD:04:51     OPN
    12 E8:28:C1:DC:C8:30     OPN
    13 E8:28:C1:DE:74:30     OPN
    14 E0:D9:E3:48:FF:D2     OPN
    15 E8:28:C1:DC:B2:41     OPN
    16 E8:28:C1:DC:B2:40     OPN
    17 00:26:99:F2:7A:E0     OPN
    18 E8:28:C1:DC:B2:42     OPN
    19 E8:28:C1:DD:04:40     OPN
    20 E8:28:C1:DD:04:41     OPN
    21 E8:28:C1:DE:47:D2     OPN
    22 02:BC:15:7E:D5:DC     OPN
    23 E8:28:C1:DC:C6:B1     OPN
    24 E8:28:C1:DD:04:42     OPN
    25 E8:28:C1:DC:C8:31     OPN
    26 E8:28:C1:DE:47:D1     OPN
    27 00:AB:0A:00:10:10     OPN
    28 E8:28:C1:DC:C6:B0     OPN
    29 E8:28:C1:DC:C6:B2     OPN
    30 E8:28:C1:DC:BD:50     OPN
    31 E8:28:C1:DC:0B:B2     OPN
    32 E8:28:C1:DC:33:12     OPN
    33 00:03:7A:1A:03:56     OPN
    34 00:03:7F:12:34:56     OPN
    35 00:3E:1A:5D:14:45     OPN
    36 E0:D9:E3:49:00:B1     OPN
    37 E8:28:C1:DC:BD:52     OPN
    38 00:26:99:F2:7A:EF     OPN
    39 02:67:F1:B0:6C:98     OPN
    40 02:CF:8B:87:B4:F9     OPN
    41 00:53:7A:99:98:56     OPN
    42 E8:28:C1:DE:47:D0     OPN

### Задание 2: Определить производителя для каждого обнаруженного устройства

-   В BSSID первые три группы MAC-адреса идентифицируют производителя
    устройства

``` r
unsafe_bssid = 
     data1 %>% 
     filter(grepl("OPN", Privacy)) %>%
     select(BSSID) %>%
     mutate(BSSID_trimmed = sub("(:[0-9A-Fa-f]{2}){3}$", "", BSSID)) %>%  
     distinct(BSSID_trimmed)

print(unsafe_bssid)
```

       BSSID_trimmed
    1       E8:28:C1
    2       00:25:00
    3       E0:D9:E3
    4       00:26:99
    5       02:BC:15
    6       00:AB:0A
    7       00:03:7A
    8       00:03:7F
    9       00:3E:1A
    10      02:67:F1
    11      02:CF:8B
    12      00:53:7A

-   Воспользуемся онлайн сервисами OUI lookup и получим следующие
    данные:

00:03:7A Taiyo Yuden Co., Ltd.

00:03:7F Atheros Communications, Inc.

00:25:00 Apple, Inc.

00:26:99 Cisco Systems, Inc

E0:D9:E3 Eltex Enterprise Ltd.

E8:28:C1 Eltex Enterprise Ltd.

Для остальных устройств совпадений в бд не найдено.

### Задание 3:Выявить устройства, использующие последнюю версию протокола шифрования WPA3, и названия точек доступа, реализованных на этих устройствах

``` r
wpa3_dev = 
  data1 %>% 
  filter(grepl("WPA3", Privacy)) %>% 
  select(BSSID, Privacy)

print(wpa3_dev)
```

                  BSSID   Privacy
    1 26:20:53:0C:98:E8 WPA3 WPA2
    2 A2:FE:FF:B8:9B:C9 WPA3 WPA2
    3 96:FF:FC:91:EF:64 WPA3 WPA2
    4 CE:48:E7:86:4E:33 WPA3 WPA2
    5 8E:1F:94:96:DA:FD WPA3 WPA2
    6 BE:FD:EF:18:92:44 WPA3 WPA2
    7 3A:DA:00:F9:0C:02 WPA3 WPA2
    8 76:C5:A0:70:08:96 WPA3 WPA2

### Задание 4: Отсортировать точки доступа по интервалу времени, в течение которого они находились на связи, по убыванию.

``` r
connection_time = 
  data1 %>%
  mutate(time = difftime(Last.time.seen, First.time.seen, units = "secs")) %>%
  arrange(desc(time)) %>%
  select(BSSID, time)
print(connection_time)
```

                    BSSID      time
    1   00:25:00:FF:94:73 9795 secs
    2   E8:28:C1:DD:04:52 9776 secs
    3   E8:28:C1:DC:B2:52 9755 secs
    4   08:3A:2F:56:35:FE 9746 secs
    5   6E:C7:EC:16:DA:1A 9729 secs
    6   E8:28:C1:DC:B2:50 9726 secs
    7   E8:28:C1:DC:B2:51 9725 secs
    8   48:5B:39:F9:7A:48 9725 secs
    9   E8:28:C1:DC:FF:F2 9724 secs
    10  8E:55:4A:85:5B:01 9723 secs
    11  00:26:99:BA:75:80 9710 secs
    12  00:26:99:F2:7A:E2 9707 secs
    13  1E:93:E3:1B:3C:F4 9633 secs
    14  9A:75:A8:B9:04:1E 9628 secs
    15  0C:80:63:A9:6E:EE 9628 secs
    16  00:23:EB:E3:81:F2 9595 secs
    17  9E:A3:A9:DB:7E:01 9555 secs
    18  E8:28:C1:DC:C8:32 9555 secs
    19  1C:7E:E5:8E:B7:DE 9524 secs
    20  00:26:99:F2:7A:E1 9492 secs
    21  BE:F1:71:D5:17:8B 9467 secs
    22  BE:F1:71:D6:10:D7 9461 secs
    23  9E:A3:A9:D6:28:3C 9451 secs
    24  E8:28:C1:DD:04:40 9400 secs
    25  E8:28:C1:DD:04:41 9400 secs
    26  00:23:EB:E3:81:F1 9348 secs
    27  00:23:EB:E3:81:FE 9305 secs
    28  00:23:EB:E3:81:FD 9305 secs
    29  9E:A3:A9:BF:12:C0 9270 secs
    30  E8:28:C1:DC:B2:40 9212 secs
    31  AA:F4:3F:EE:49:0B 9045 secs
    32  E8:28:C1:DE:47:D2 9041 secs
    33  E8:28:C1:DD:04:50 8989 secs
    34  14:EB:B6:6A:76:37 8915 secs
    35  56:99:98:EE:5A:4E 8811 secs
    36  E8:28:C1:DC:B2:42 8693 secs
    37  38:1A:52:0D:90:A1 8661 secs
    38  0A:C5:E1:DB:17:7B 8608 secs
    39  E8:28:C1:DC:C8:30 8445 secs
    40  E8:28:C1:DC:C6:B1 8390 secs
    41  E8:28:C1:DD:04:42 8318 secs
    42  E8:28:C1:DC:B2:41 8307 secs
    43  12:51:07:FF:29:D6 7483 secs
    44  CE:B3:FF:84:45:FC 7271 secs
    45  E8:28:C1:DC:C8:31 7199 secs
    46  E8:28:C1:DC:C6:B2 6819 secs
    47  4A:EC:1E:DB:BF:95 6658 secs
    48  00:26:99:F2:7A:E0 6218 secs
    49  E8:28:C1:DD:04:51 5643 secs
    50  E0:D9:E3:48:FF:D2 5624 secs
    51  00:AB:0A:00:10:10 5356 secs
    52  E8:28:C1:DE:74:32 5190 secs
    53  10:50:72:00:11:08 4997 secs
    54  EA:D8:D1:77:C8:08 4995 secs
    55  D2:6D:52:61:51:5D 4636 secs
    56  E0:D9:E3:49:04:52 4614 secs
    57  7E:3A:10:A7:59:4E 4611 secs
    58  BE:F1:71:D5:0E:53 4578 secs
    59  A6:02:B9:73:83:18 4577 secs
    60  9A:9F:06:44:24:5B 4572 secs
    61  E8:28:C1:DE:74:31 4433 secs
    62  92:F5:7B:43:0B:69 4392 secs
    63  E8:28:C1:DC:3C:92 4331 secs
    64  38:1A:52:0D:84:D7 4319 secs
    65  38:1A:52:0D:90:5D 4255 secs
    66  A2:64:E8:97:58:EE 4252 secs
    67  A6:02:B9:73:81:47 4224 secs
    68  56:C5:2B:9F:84:90 4173 secs
    69  A6:02:B9:73:2F:76 4144 secs
    70  38:1A:52:0D:97:60 4086 secs
    71  0A:24:D8:D9:24:70 4071 secs
    72  E8:28:C1:DC:C6:B0 3879 secs
    73  8A:A3:03:73:52:08 3451 secs
    74  5E:C7:C0:E4:D7:D4 3265 secs
    75  E8:28:C1:DC:54:72 3074 secs
    76  4A:86:77:04:B7:28 3008 secs
    77  B6:C4:55:B5:53:24 2987 secs
    78  E8:28:C1:DC:BD:50 2743 secs
    79  76:70:AF:A4:D2:AF 2733 secs
    80  86:DF:BF:E4:2F:23 2688 secs
    81  38:1A:52:0D:8F:EC 2635 secs
    82  EA:7B:9B:D8:56:34 2241 secs
    83  38:1A:52:0D:85:1D 2082 secs
    84  00:26:CB:AA:62:71 1969 secs
    85  96:FF:FC:91:EF:64 1928 secs
    86  E8:28:C1:DC:33:12 1379 secs
    87  E8:28:C1:DC:F0:90 1312 secs
    88  3A:70:96:C6:30:2C 1300 secs
    89  36:46:53:81:12:A0 1248 secs
    90  CE:C3:F7:A4:7E:B3 1224 secs
    91  26:20:53:0C:98:E8 1045 secs
    92  92:12:38:E5:7E:1E  868 secs
    93  E8:28:C1:DC:33:10  846 secs
    94  E8:28:C1:DB:F5:F0  842 secs
    95  E8:28:C1:DC:0B:B0  832 secs
    96  E8:28:C1:DB:F5:F2  782 secs
    97  02:67:F1:B0:6C:98  651 secs
    98  E8:28:C1:DE:74:30  508 secs
    99  1E:C2:8E:D8:30:91  498 secs
    100 8E:1F:94:96:DA:FD  415 secs
    101 E0:D9:E3:49:04:50  401 secs
    102 CE:48:E7:86:4E:33  295 secs
    103 00:26:99:BA:75:8F  288 secs
    104 2A:E8:A2:02:01:73  220 secs
    105 2E:FE:13:D0:96:51   58 secs
    106 9C:A5:13:28:D5:89   43 secs
    107 22:C9:7F:A9:BA:9C   41 secs
    108 E8:28:C1:DC:54:B0   36 secs
    109 D2:25:91:F6:6C:D8   13 secs
    110 3A:DA:00:F9:0C:02    9 secs
    111 E8:28:C1:DB:FC:F2    9 secs
    112 DC:09:4C:32:34:9B    8 secs
    113 F2:30:AB:E9:03:ED    7 secs
    114 E0:D9:E3:49:04:40    7 secs
    115 00:03:7A:1A:03:56    6 secs
    116 B2:CF:C0:00:4A:60    5 secs
    117 BE:FD:EF:18:92:44    4 secs
    118 02:BC:15:7E:D5:DC    2 secs
    119 00:23:EB:E3:49:31    2 secs
    120 00:3E:1A:5D:14:45    2 secs
    121 76:C5:A0:70:08:96    2 secs
    122 82:CD:7D:04:17:3B    2 secs
    123 E0:D9:E3:49:00:B0    1 secs
    124 E8:28:C1:DC:54:B2    1 secs
    125 C6:BC:37:7A:67:0D    0 secs
    126 12:48:F9:CF:58:8E    0 secs
    127 76:E4:ED:B0:5C:9A    0 secs
    128 E0:D9:E3:48:FF:D0    0 secs
    129 E2:37:BF:8F:6A:7B    0 secs
    130 C2:B5:D7:7F:07:A8    0 secs
    131 8A:4E:75:44:5A:F6    0 secs
    132 00:03:7A:1A:18:56    0 secs
    133 E8:28:C1:DE:47:D1    0 secs
    134 A2:FE:FF:B8:9B:C9    0 secs
    135 00:09:9A:12:55:04    0 secs
    136 E8:28:C1:DC:3A:B0    0 secs
    137 E8:28:C1:DC:0B:B2    0 secs
    138 E8:28:C1:DC:3C:80    0 secs
    139 00:23:EB:E3:44:31    0 secs
    140 A6:F7:05:31:E8:EE    0 secs
    141 BA:2A:7A:DD:38:3E    0 secs
    142 12:54:1A:C6:FF:71    0 secs
    143 76:5E:F3:F9:A5:1C    0 secs
    144 00:03:7F:12:34:56    0 secs
    145 E8:28:C1:DC:03:30    0 secs
    146 B2:1B:0C:67:0A:BD    0 secs
    147 E0:D9:E3:49:00:B1    0 secs
    148 E8:28:C1:DC:BD:52    0 secs
    149 E8:28:C1:DE:72:D0    0 secs
    150 E0:D9:E3:49:04:41    0 secs
    151 00:26:99:F1:1A:E1    0 secs
    152 00:23:EB:E3:44:32    0 secs
    153 00:26:CB:AA:62:72    0 secs
    154 E0:D9:E3:48:B4:D2    0 secs
    155 AE:3E:7F:C8:BC:8E    0 secs
    156 02:B3:45:5A:05:93    0 secs
    157 00:00:00:00:00:00    0 secs
    158 6A:B0:1A:C2:DF:49    0 secs
    159 E8:28:C1:DC:3C:90    0 secs
    160 30:B4:B8:11:C0:90    0 secs
    161 00:26:99:F2:7A:EF    0 secs
    162 02:CF:8B:87:B4:F9    0 secs
    163 E8:28:C1:DC:03:32    0 secs
    164 00:53:7A:99:98:56    0 secs
    165 00:03:7F:10:17:56    0 secs
    166 00:0D:97:6B:93:DF    0 secs
    167 E8:28:C1:DE:47:D0    0 secs

### Задание 5: Обнаружить топ-10 самых быстрых точек доступа.

``` r
fast_points = 
   data1 %>% 
   arrange(desc(Speed)) %>% slice(1:10)  %>% 
   select(BSSID, Speed)
print(fast_points)
```

                   BSSID Speed
    1  26:20:53:0C:98:E8   866
    2  96:FF:FC:91:EF:64   866
    3  CE:48:E7:86:4E:33   866
    4  8E:1F:94:96:DA:FD   866
    5  9A:75:A8:B9:04:1E   360
    6  4A:EC:1E:DB:BF:95   360
    7  56:C5:2B:9F:84:90   360
    8  E8:28:C1:DC:B2:41   360
    9  E8:28:C1:DC:B2:40   360
    10 E8:28:C1:DC:B2:42   360

### Задание 6: Отсортировать точки доступа по частоте отправки запросов (beacons) в единицу времени по их убыванию.

``` r
beacons_rate = 
  data1 %>%
  mutate(beacons_rate =  X..beacons / as.numeric(difftime(Last.time.seen,  First.time.seen, units = "secs"))) %>%  
  filter(!is.infinite(beacons_rate) & !is.nan(beacons_rate)) %>%
  arrange(desc(beacons_rate)) %>% 
  select(BSSID, beacons_rate)
print(beacons_rate)
```

                    BSSID beacons_rate
    1   F2:30:AB:E9:03:ED 0.8571428571
    2   B2:CF:C0:00:4A:60 0.8000000000
    3   3A:DA:00:F9:0C:02 0.5555555556
    4   02:BC:15:7E:D5:DC 0.5000000000
    5   00:3E:1A:5D:14:45 0.5000000000
    6   76:C5:A0:70:08:96 0.5000000000
    7   D2:25:91:F6:6C:D8 0.3846153846
    8   BE:F1:71:D6:10:D7 0.1740830779
    9   00:03:7A:1A:03:56 0.1666666667
    10  38:1A:52:0D:84:D7 0.1630006946
    11  0A:C5:E1:DB:17:7B 0.1453299257
    12  1E:93:E3:1B:3C:F4 0.1442956504
    13  D2:6D:52:61:51:5D 0.1395599655
    14  BE:F1:71:D5:0E:53 0.1347750109
    15  4A:86:77:04:B7:28 0.1200132979
    16  3A:70:96:C6:30:2C 0.1115384615
    17  76:70:AF:A4:D2:AF 0.0925722649
    18  BE:F1:71:D5:17:8B 0.0893630506
    19  AA:F4:3F:EE:49:0B 0.0815920398
    20  6E:C7:EC:16:DA:1A 0.0770891150
    21  4A:EC:1E:DB:BF:95 0.0765995795
    22  56:C5:2B:9F:84:90 0.0759645339
    23  9A:75:A8:B9:04:1E 0.0720814292
    24  9C:A5:13:28:D5:89 0.0697674419
    25  36:46:53:81:12:A0 0.0657051282
    26  38:1A:52:0D:85:1D 0.0624399616
    27  38:1A:52:0D:8F:EC 0.0406072106
    28  2E:FE:13:D0:96:51 0.0344827586
    29  CE:48:E7:86:4E:33 0.0305084746
    30  8E:1F:94:96:DA:FD 0.0289156627
    31  E8:28:C1:DC:B2:51 0.0286889460
    32  E8:28:C1:DC:B2:50 0.0267324697
    33  5E:C7:C0:E4:D7:D4 0.0260336907
    34  E8:28:C1:DC:B2:52 0.0257303947
    35  8E:55:4A:85:5B:01 0.0255065309
    36  38:1A:52:0D:90:5D 0.0211515864
    37  1C:7E:E5:8E:B7:DE 0.0149097018
    38  38:1A:52:0D:90:A1 0.0129315322
    39  A2:64:E8:97:58:EE 0.0122295390
    40  1E:C2:8E:D8:30:91 0.0120481928
    41  48:5B:39:F9:7A:48 0.0112082262
    42  00:26:99:F2:7A:E2 0.0086535490
    43  38:1A:52:0D:97:60 0.0068526676
    44  00:26:99:F2:7A:E1 0.0068478719
    45  00:26:99:BA:75:80 0.0062821833
    46  A6:02:B9:73:2F:76 0.0062741313
    47  9E:A3:A9:D6:28:3C 0.0053962544
    48  00:23:EB:E3:81:FE 0.0050510478
    49  00:23:EB:E3:81:FD 0.0049435787
    50  9A:9F:06:44:24:5B 0.0048118985
    51  96:FF:FC:91:EF:64 0.0046680498
    52  A6:02:B9:73:81:47 0.0044981061
    53  0C:80:63:A9:6E:EE 0.0043622767
    54  12:51:07:FF:29:D6 0.0042763597
    55  9E:A3:A9:DB:7E:01 0.0041862899
    56  92:F5:7B:43:0B:69 0.0040983607
    57  86:DF:BF:E4:2F:23 0.0040922619
    58  A6:02:B9:73:83:18 0.0037142233
    59  E8:28:C1:DD:04:40 0.0031914894
    60  26:20:53:0C:98:E8 0.0028708134
    61  E8:28:C1:DD:04:42 0.0027650878
    62  E8:28:C1:DD:04:41 0.0026595745
    63  B6:C4:55:B5:53:24 0.0023434884
    64  E8:28:C1:DD:04:50 0.0022249416
    65  00:23:EB:E3:81:F1 0.0020325203
    66  E8:28:C1:DC:BD:50 0.0018228217
    67  E8:28:C1:DD:04:51 0.0015948963
    68  02:67:F1:B0:6C:98 0.0015360983
    69  E8:28:C1:DC:C8:32 0.0012558870
    70  E8:28:C1:DC:C8:31 0.0011112655
    71  E8:28:C1:DC:C6:B0 0.0010311936
    72  00:26:CB:AA:62:71 0.0010157440
    73  9E:A3:A9:BF:12:C0 0.0009708738
    74  E8:28:C1:DC:C8:30 0.0008288928
    75  00:23:EB:E3:81:F2 0.0007295466
    76  7E:3A:10:A7:59:4E 0.0006506181
    77  E8:28:C1:DC:B2:41 0.0006019020
    78  E8:28:C1:DC:C6:B1 0.0005959476
    79  E8:28:C1:DC:B2:42 0.0005751754
    80  E8:28:C1:DC:B2:40 0.0005427703
    81  0A:24:D8:D9:24:70 0.0004912798
    82  E8:28:C1:DE:74:31 0.0004511617
    83  EA:7B:9B:D8:56:34 0.0004462294
    84  E8:28:C1:DD:04:52 0.0004091653
    85  10:50:72:00:11:08 0.0004002401
    86  E8:28:C1:DE:47:D2 0.0003318217
    87  EA:D8:D1:77:C8:08 0.0002002002
    88  E8:28:C1:DE:74:32 0.0001926782
    89  56:99:98:EE:5A:4E 0.0001134945
    90  E8:28:C1:DC:FF:F2 0.0000000000
    91  00:25:00:FF:94:73 0.0000000000
    92  08:3A:2F:56:35:FE 0.0000000000
    93  E8:28:C1:DE:74:30 0.0000000000
    94  E0:D9:E3:48:FF:D2 0.0000000000
    95  00:26:99:F2:7A:E0 0.0000000000
    96  2A:E8:A2:02:01:73 0.0000000000
    97  E8:28:C1:DC:3C:92 0.0000000000
    98  14:EB:B6:6A:76:37 0.0000000000
    99  CE:B3:FF:84:45:FC 0.0000000000
    100 E8:28:C1:DC:54:72 0.0000000000
    101 00:AB:0A:00:10:10 0.0000000000
    102 E8:28:C1:DC:C6:B2 0.0000000000
    103 E8:28:C1:DB:F5:F2 0.0000000000
    104 BE:FD:EF:18:92:44 0.0000000000
    105 00:23:EB:E3:49:31 0.0000000000
    106 CE:C3:F7:A4:7E:B3 0.0000000000
    107 E8:28:C1:DC:33:12 0.0000000000
    108 E8:28:C1:DB:FC:F2 0.0000000000
    109 00:26:99:BA:75:8F 0.0000000000
    110 DC:09:4C:32:34:9B 0.0000000000
    111 E8:28:C1:DC:F0:90 0.0000000000
    112 E0:D9:E3:49:04:52 0.0000000000
    113 E0:D9:E3:49:04:50 0.0000000000
    114 E0:D9:E3:49:04:40 0.0000000000
    115 E8:28:C1:DC:54:B0 0.0000000000
    116 E0:D9:E3:49:00:B0 0.0000000000
    117 E8:28:C1:DC:33:10 0.0000000000
    118 E8:28:C1:DB:F5:F0 0.0000000000
    119 8A:A3:03:73:52:08 0.0000000000
    120 22:C9:7F:A9:BA:9C 0.0000000000
    121 92:12:38:E5:7E:1E 0.0000000000
    122 E8:28:C1:DC:0B:B0 0.0000000000
    123 82:CD:7D:04:17:3B 0.0000000000
    124 E8:28:C1:DC:54:B2 0.0000000000

## Данные клиентов

### Задание 1: Определить производителя для каждого обнаруженного устройства

-   В BSSID первые три группы MAC-адреса идентифицируют производителя
    устройства. Также отфильтруем значения не в формате MAC-адреса

``` r
user_bssid =
  data2 %>%
  filter(BSSID != '(not associated)') %>%
  filter(grepl("^[0-9A-Fa-f]{2}(:[0-9A-Fa-f]{2}){5}$", BSSID)) %>%  
  mutate(BSSID_trimmed = substr(BSSID, 1, 8)) %>%
  select(BSSID_trimmed)
print(unique(user_bssid))
```

        BSSID_trimmed
    1        BE:F1:71
    4        1E:93:E3
    5        E8:28:C1
    6        00:25:00
    7        00:26:99
    8        0C:80:63
    10       0A:C5:E1
    12       9A:75:A8
    13       8A:A3:03
    14       4A:EC:1E
    16       08:3A:2F
    17       6E:C7:EC
    21       2A:E8:A2
    28       56:C5:2B
    30       9A:9F:06
    31       12:48:F9
    35       AA:F4:3F
    37       3A:70:96
    42       8E:55:4A
    43       5E:C7:C0
    44       E2:37:BF
    48       96:FF:FC
    50       CE:B3:FF
    58       76:70:AF
    60       00:AB:0A
    65       8E:1F:94
    77       EA:7B:9B
    78       BE:FD:EF
    80       7E:3A:10
    82       00:23:EB
    86       E0:D9:E3
    87       3A:DA:00
    99       92:F5:7B
    102      DC:09:4C
    108      22:C9:7F
    114      92:12:38
    117      B2:1B:0C
    131      1E:C2:8E
    133      A2:64:E8
    135      A6:02:B9
    147      AE:3E:7F
    155      B6:C4:55
    158      86:DF:BF
    160      02:67:F1
    166      36:46:53
    173      82:CD:7D
    179      00:03:7F
    180      00:0D:97

-   Воспользуемся онлайн сервисами OUI lookup и получим следующие
    данные:

00:03:7F Atheros Communications, Inc.

00:0D:97 Hitachi Energy USA Inc.

00:23:EB Cisco Systems, Inc

00:25:00 Apple, Inc.

00:26:99 Cisco Systems, Inc

08:3A:2F Guangzhou Juan Intelligent Tech Joint Stock Co.,Ltd

0C:80:63 Tp-Link Technologies Co.,Ltd.

DC:09:4C Huawei Technologies Co.,Ltd

E0:D9:E3 Eltex Enterprise Ltd.

E8:28:C1 Eltex Enterprise Ltd.

Для остальных устройств совпадений в бд не найдено.

### Задание 2: Обнаружить устройства, которые НЕ рандомизируют свой MAC адрес

-   Обычно устройства, которые рандомизируют свои MAC-адреса, будут
    показывать различные значения в разных сессиях или при повторных
    сканированиях.

``` r
non_randomized_devices =
  data2 %>%
  filter(BSSID != '(not associated)') %>%
  filter(grepl("^[0-9A-Fa-f]{2}(:[0-9A-Fa-f]{2}){5}$", BSSID)) %>%
  group_by(BSSID) %>%
  summarise(count = n()) %>%
  filter(count == 1) %>%  
  select(BSSID)
print(unique(non_randomized_devices))
```

    # A tibble: 51 × 1
       BSSID            
       <chr>            
     1 00:03:7F:10:17:56
     2 00:0D:97:6B:93:DF
     3 00:23:EB:E3:49:31
     4 00:26:99:F2:7A:E1
     5 00:AB:0A:00:10:10
     6 02:67:F1:B0:6C:98
     7 0A:C5:E1:DB:17:7B
     8 12:48:F9:CF:58:8E
     9 1E:C2:8E:D8:30:91
    10 22:C9:7F:A9:BA:9C
    # ℹ 41 more rows

### Задание 3: Кластеризовать запросы от устройств к точкам доступа по их именам. Определить время появления устройства в зоне радиовидимости и время выхода его из нее.

``` r
device_visibility =
  data2 %>% 
  group_by(Probed.ESSIDs) %>% 
  summarise(
  first_seen = min(First.time.seen, na.rm = TRUE),
  last_seen = max(Last.time.seen, na.rm = TRUE),
  .groups = 'drop' )
print(device_visibility)
```

    # A tibble: 108 × 3
       Probed.ESSIDs                first_seen          last_seen          
       <chr>                        <dttm>              <dttm>             
     1 -D-13-                       2023-07-28 09:14:42 2023-07-28 10:26:42
     2 1                            2023-07-28 10:36:12 2023-07-28 11:56:13
     3 107                          2023-07-28 10:29:43 2023-07-28 10:29:43
     4 531                          2023-07-28 10:57:04 2023-07-28 10:57:04
     5 AAAAAOB/CC0ADwGkRedmi 3S     2023-07-28 09:34:20 2023-07-28 11:44:40
     6 AKADO-D967                   2023-07-28 10:31:55 2023-07-28 10:31:55
     7 AQAAAB6zaIoATwEURedmi Note 5 2023-07-28 10:25:19 2023-07-28 11:51:48
     8 ASUS                         2023-07-28 10:31:13 2023-07-28 10:31:13
     9 Alex-net2                    2023-07-28 10:01:06 2023-07-28 10:01:06
    10 AndroidAP177B                2023-07-28 09:13:09 2023-07-28 11:34:42
    # ℹ 98 more rows

### Задание 4: Оценить стабильность уровня сигнала внури кластера во времени. Выявить наиболее стабильный кластер.

-   Для оценки стабильности оценить математическое ожидание и
    среднеквадратичное отклонение для каждого найденного кластера.
-   Преобразуем столбец power в числовые значения для оценки

``` r
data2$Power <- as.numeric(as.character(data2$Power))
```

    Warning: в результате преобразования созданы NA

``` r
na_values <- data2 %>% filter(is.na(Power))

signal_stability <- data2 %>%
  filter(!is.na(Probed.ESSIDs), !is.na(Power)) %>%  
  group_by(Probed.ESSIDs) %>%
  summarise(
    mean_signal = mean(Power, na.rm = TRUE),  
    sd_signal = sd(Power, na.rm = TRUE),      
    .groups = 'drop'  
  ) %>%
  arrange(sd_signal, mean_signal)  


most_stable_cluster <- signal_stability %>%
  slice(1)  

print(signal_stability)
```

    # A tibble: 107 × 3
       Probed.ESSIDs  mean_signal sd_signal
       <chr>                <dbl>     <dbl>
     1 Galaxy A71           -48.5     0.707
     2 Kesha                -52       1.41 
     3 podval               -65       1.63 
     4 Tupik                -65.7     2.31 
     5 KB-12                -69       2.83 
     6 Reconn-Guest         -69       2.83 
     7 MT_FREE              -65.4     2.97 
     8 OKB                  -71.7     3.06 
     9 Rayskaya_banya       -68.3     3.06 
    10 IKB                  -55.7     3.06 
    # ℹ 97 more rows

``` r
print(most_stable_cluster)
```

    # A tibble: 1 × 3
      Probed.ESSIDs mean_signal sd_signal
      <chr>               <dbl>     <dbl>
    1 Galaxy A71          -48.5     0.707

## Оценка результата

С использованием инструментов tidyverse и RStudio были проделаны задания
по основам обработки данных.

## Вывод

Провели работу с tidyverse, данными p2_wif_data, повторили технологии
подготовки и анализа данных.
