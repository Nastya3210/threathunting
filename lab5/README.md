# Исследование информации о состоянии беспроводных сетей
nastya5908@yandex.ru

## Цель работы

1.  Получить знания о методах исследования радиоэлектронной обстановки.
2.  Составить представление о механизмах работы Wi-Fi сетей на канальном
    и сетевом уровне модели OSI.
3.  Закрепить практические навыки использования языка программирования R
    для обработки данных
4.  Закрепить знания основных функций обработки данных экосистемы
    tidyverse языка R

## Исходные данные

1.  Программное обеспечение Windows 11
2.  Rstudio Desktop
3.  Интерпретатор языка R 4.5.1

## План:

1.  Импортировать и подготовить данные

2.  Проанализировать данные

3.  Составить отчет и выложить его и исходный qmd/rmd файл в свой
    репозиторий

## Шаги

Установим и подключим необходимые библиотеки

``` r
library(dplyr)
```


    Присоединяю пакет: 'dplyr'

    Следующие объекты скрыты от 'package:stats':

        filter, lag

    Следующие объекты скрыты от 'package:base':

        intersect, setdiff, setequal, union

``` r
library(tidyverse)
```

    Warning: пакет 'ggplot2' был собран под R версии 4.5.2

    ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ✔ forcats   1.0.1     ✔ readr     2.1.5
    ✔ ggplot2   4.0.1     ✔ stringr   1.5.2
    ✔ lubridate 1.9.4     ✔ tibble    3.3.0
    ✔ purrr     1.1.0     ✔ tidyr     1.3.1

    ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ✖ dplyr::filter() masks stats::filter()
    ✖ dplyr::lag()    masks stats::lag()
    ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

``` r
library(lubridate)
library(httr)
```

    Warning: пакет 'httr' был собран под R версии 4.5.2

``` r
library(jsonlite)
```


    Присоединяю пакет: 'jsonlite'

    Следующий объект скрыт от 'package:purrr':

        flatten

#### 1. Подготовка данных

1\. Импортируйте данные

``` r
lines <- read_lines("P2_wifi_data.csv")
start <- which(grepl("Station MAC", lines, fixed = TRUE))
td_data <- read_csv("P2_wifi_data.csv", n_max = start - 4)
```

    Rows: 167 Columns: 15
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr  (6): BSSID, Privacy, Cipher, Authentication, LAN IP, ESSID
    dbl  (6): channel, Speed, Power, # beacons, # IV, ID-length
    lgl  (1): Key
    dttm (2): First time seen, Last time seen

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
client_data <- read_csv("P2_wifi_data.csv", skip = start - 1)
```

    Warning: One or more parsing issues, call `problems()` on your data frame for details,
    e.g.:
      dat <- vroom(...)
      problems(dat)

    Rows: 12081 Columns: 7
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr  (3): Station MAC, BSSID, Probed ESSIDs
    dbl  (2): Power, # packets
    dttm (2): First time seen, Last time seen

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

2\. Привести датасеты в вид “аккуратных данных”, преобразовать типы
столбцов в соответствии с типом данных

``` r
td_clean <- td_data |>
    rename(
    first_seen = `First time seen`,
    last_seen = `Last time seen`,
    speed = Speed,
    privacy = Privacy,
    cipher = Cipher,
    auth = Authentication,
    power = Power,
    beacons = `# beacons`,
    iv = `# IV`,
    lan_ip = `LAN IP`,
    id_length = `ID-length`,
    essid = ESSID,
    key = Key
  ) |>
  mutate(
    first_seen = as_datetime(first_seen),
    last_seen = as_datetime(last_seen),
    channel = as.integer(channel),
    speed = as.integer(speed),
    power = as.integer(power),
    beacons = as.integer(beacons),
    iv = as.integer(iv),
    id_length = as.integer(id_length)) |>
    filter(!is.na(BSSID))
```

``` r
client_clean <- client_data |>
    rename(
    station_mac = `Station MAC`,
    first_seen = `First time seen`,
    last_seen = `Last time seen`,
    power = Power,
    packets = `# packets`,
    probed_ESSIDs = `Probed ESSIDs`
  ) |>
  mutate(
    first_seen = as_datetime(first_seen),
    last_seen = as_datetime(last_seen),
    power = as.integer(power),
    packets = as.integer(packets)) |>
    filter(!is.na(BSSID))
```

3\. Просмотрите общую структуру данных с помощью функции glimpse()

``` r
glimpse(td_clean)
```

    Rows: 167
    Columns: 15
    $ BSSID      <chr> "BE:F1:71:D5:17:8B", "6E:C7:EC:16:DA:1A", "9A:75:A8:B9:04:1…
    $ first_seen <dttm> 2023-07-28 09:13:03, 2023-07-28 09:13:03, 2023-07-28 09:13…
    $ last_seen  <dttm> 2023-07-28 11:50:50, 2023-07-28 11:55:12, 2023-07-28 11:53…
    $ channel    <int> 1, 1, 1, 7, 6, 6, 11, 11, 11, 1, 6, 14, 11, 11, 6, 6, 6, 6,…
    $ speed      <int> 195, 130, 360, 360, 130, 130, 195, 130, 130, 195, 180, 65, …
    $ privacy    <chr> "WPA2", "WPA2", "WPA2", "WPA2", "WPA2", "OPN", "WPA2", "WPA…
    $ cipher     <chr> "CCMP", "CCMP", "CCMP", "CCMP", "CCMP", NA, "CCMP", "CCMP",…
    $ auth       <chr> "PSK", "PSK", "PSK", "PSK", "PSK", NA, "PSK", "PSK", "PSK",…
    $ power      <int> -30, -30, -68, -37, -57, -63, -27, -38, -38, -66, -42, -62,…
    $ beacons    <int> 846, 750, 694, 510, 647, 251, 1647, 1251, 704, 617, 1390, 1…
    $ iv         <int> 504, 116, 26, 21, 6, 3430, 80, 11, 0, 0, 86, 0, 0, 0, 907, …
    $ lan_ip     <chr> "0.  0.  0.  0", "0.  0.  0.  0", "0.  0.  0.  0", "0.  0. …
    $ id_length  <int> 12, 4, 2, 14, 25, 13, 12, 13, 24, 12, 10, 0, 24, 24, 12, 0,…
    $ essid      <chr> "C322U13 3965", "Cnet", "KC", "POCO X5 Pro 5G", NA, "MIREA_…
    $ key        <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA,…

``` r
glimpse(client_clean)
```

    Rows: 12,081
    Columns: 7
    $ station_mac   <chr> "CA:66:3B:8F:56:DD", "96:35:2D:3D:85:E6", "5C:3A:45:9E:1…
    $ first_seen    <dttm> 2023-07-28 09:13:03, 2023-07-28 09:13:03, 2023-07-28 09…
    $ last_seen     <dttm> 2023-07-28 10:59:44, 2023-07-28 09:13:03, 2023-07-28 11…
    $ power         <int> -33, -65, -39, -61, -53, -43, -31, -71, -74, -65, -45, -…
    $ packets       <int> 858, 4, 432, 958, 1, 344, 163, 3, 115, 437, 265, 77, 7, …
    $ BSSID         <chr> "BE:F1:71:D5:17:8B", "(not associated)", "BE:F1:71:D6:10…
    $ probed_ESSIDs <chr> "C322U13 3965", "IT2 Wireless", "C322U21 0566", "C322U13…

#### 2. Анализ

1\. Определить небезопасные точки доступа (без шифрования – OPN)

``` r
nebez_td <- td_clean |> filter(privacy == "OPN")
print(nebez_td)
```

    # A tibble: 42 × 15
       BSSID    first_seen          last_seen           channel speed privacy cipher
       <chr>    <dttm>              <dttm>                <int> <int> <chr>   <chr> 
     1 E8:28:C… 2023-07-28 09:13:03 2023-07-28 11:55:38       6   130 OPN     <NA>  
     2 E8:28:C… 2023-07-28 09:13:06 2023-07-28 11:55:12       6   130 OPN     <NA>  
     3 E8:28:C… 2023-07-28 09:13:06 2023-07-28 11:55:11       6   130 OPN     <NA>  
     4 E8:28:C… 2023-07-28 09:13:06 2023-07-28 11:55:10       6    -1 OPN     <NA>  
     5 00:25:0… 2023-07-28 09:13:06 2023-07-28 11:56:21      44    -1 OPN     <NA>  
     6 E8:28:C… 2023-07-28 09:13:09 2023-07-28 11:56:05      11   130 OPN     <NA>  
     7 E8:28:C… 2023-07-28 09:13:13 2023-07-28 10:27:06       6   130 OPN     <NA>  
     8 E8:28:C… 2023-07-28 09:13:13 2023-07-28 10:39:43       6   130 OPN     <NA>  
     9 E8:28:C… 2023-07-28 09:13:17 2023-07-28 11:52:32       1   130 OPN     <NA>  
    10 E8:28:C… 2023-07-28 09:13:50 2023-07-28 11:43:39      11   130 OPN     <NA>  
    # ℹ 32 more rows
    # ℹ 8 more variables: auth <chr>, power <int>, beacons <int>, iv <int>,
    #   lan_ip <chr>, id_length <int>, essid <chr>, key <lgl>

2\. Определить производителя для каждого обнаруженного устройства

``` r
get_mac <- function(mac) {
  url <- paste0("https://api.macvendors.com/", mac)
  Sys.sleep(1)
  tryCatch({
    response <- GET(url)
    if (status_code(response) == 200) {
      vendor <- content(response, "text", encoding = "UTF-8")
      return(vendor)
    } else {return(NA)}})}

nebez_td <- nebez_td |>
  rowwise() |>
  mutate(manufacturer = get_mac(BSSID)) |>
  ungroup()
```

``` r
nebez_td |>
     select(BSSID, manufacturer) |>
     print(n=Inf) 
```

    # A tibble: 42 × 2
       BSSID             manufacturer
       <chr>             <lgl>       
     1 E8:28:C1:DC:B2:52 NA          
     2 E8:28:C1:DC:B2:50 NA          
     3 E8:28:C1:DC:B2:51 NA          
     4 E8:28:C1:DC:FF:F2 NA          
     5 00:25:00:FF:94:73 NA          
     6 E8:28:C1:DD:04:52 NA          
     7 E8:28:C1:DE:74:31 NA          
     8 E8:28:C1:DE:74:32 NA          
     9 E8:28:C1:DC:C8:32 NA          
    10 E8:28:C1:DD:04:50 NA          
    11 E8:28:C1:DD:04:51 NA          
    12 E8:28:C1:DC:C8:30 NA          
    13 E8:28:C1:DE:74:30 NA          
    14 E0:D9:E3:48:FF:D2 NA          
    15 E8:28:C1:DC:B2:41 NA          
    16 E8:28:C1:DC:B2:40 NA          
    17 00:26:99:F2:7A:E0 NA          
    18 E8:28:C1:DC:B2:42 NA          
    19 E8:28:C1:DD:04:40 NA          
    20 E8:28:C1:DD:04:41 NA          
    21 E8:28:C1:DE:47:D2 NA          
    22 02:BC:15:7E:D5:DC NA          
    23 E8:28:C1:DC:C6:B1 NA          
    24 E8:28:C1:DD:04:42 NA          
    25 E8:28:C1:DC:C8:31 NA          
    26 E8:28:C1:DE:47:D1 NA          
    27 00:AB:0A:00:10:10 NA          
    28 E8:28:C1:DC:C6:B0 NA          
    29 E8:28:C1:DC:C6:B2 NA          
    30 E8:28:C1:DC:BD:50 NA          
    31 E8:28:C1:DC:0B:B2 NA          
    32 E8:28:C1:DC:33:12 NA          
    33 00:03:7A:1A:03:56 NA          
    34 00:03:7F:12:34:56 NA          
    35 00:3E:1A:5D:14:45 NA          
    36 E0:D9:E3:49:00:B1 NA          
    37 E8:28:C1:DC:BD:52 NA          
    38 00:26:99:F2:7A:EF NA          
    39 02:67:F1:B0:6C:98 NA          
    40 02:CF:8B:87:B4:F9 NA          
    41 00:53:7A:99:98:56 NA          
    42 E8:28:C1:DE:47:D0 NA          

3\. Выявить устройства, использующие последнюю версию протокола
шифрования WPA3, и названия точек доступа, реализованных на этих
устройствах

``` r
td_clean |> filter(str_detect(privacy, "WPA3")) |> select(BSSID, essid, privacy)
```

    # A tibble: 8 × 3
      BSSID             essid                                          privacy  
      <chr>             <chr>                                          <chr>    
    1 26:20:53:0C:98:E8  <NA>                                          WPA3 WPA2
    2 A2:FE:FF:B8:9B:C9 "Christie’s"                                   WPA3 WPA2
    3 96:FF:FC:91:EF:64  <NA>                                          WPA3 WPA2
    4 CE:48:E7:86:4E:33 "iPhone (Анастасия)"                           WPA3 WPA2
    5 8E:1F:94:96:DA:FD "iPhone (Анастасия)"                           WPA3 WPA2
    6 BE:FD:EF:18:92:44 "Димасик"                                      WPA3 WPA2
    7 3A:DA:00:F9:0C:02 "iPhone XS Max \U0001f98a\U0001f431\U0001f98a" WPA3 WPA2
    8 76:C5:A0:70:08:96  <NA>                                          WPA3 WPA2

4\. Отсортировать точки доступа по интервалу времени, в течение которого
они находились на связи, по убыванию.

``` r
td_sessions <- td_clean |> arrange(BSSID, first_seen) |> group_by(BSSID) |>
  mutate( gap_seconds = as.numeric(
      difftime(first_seen, lag(last_seen, default = first(first_seen)))),
      session_id = cumsum(gap_seconds > 2700) + 1) |>
  
  group_by(BSSID, session_id) |>
  summarise(
    start = min(first_seen),
    end = max(last_seen),
    .groups = 'drop')|>
  
  group_by(BSSID) |> mutate(sessions = n()) |> ungroup() |>
  mutate(dlit = as.numeric(end - start, units = "secs")) |>
  arrange(desc(dlit)) |> select(BSSID, sessions, start, end, dlit)

td_sessions
```

    # A tibble: 167 × 5
       BSSID             sessions start               end                  dlit
       <chr>                <int> <dttm>              <dttm>              <dbl>
     1 00:25:00:FF:94:73        1 2023-07-28 09:13:06 2023-07-28 11:56:21  9795
     2 E8:28:C1:DD:04:52        1 2023-07-28 09:13:09 2023-07-28 11:56:05  9776
     3 E8:28:C1:DC:B2:52        1 2023-07-28 09:13:03 2023-07-28 11:55:38  9755
     4 08:3A:2F:56:35:FE        1 2023-07-28 09:13:27 2023-07-28 11:55:53  9746
     5 6E:C7:EC:16:DA:1A        1 2023-07-28 09:13:03 2023-07-28 11:55:12  9729
     6 E8:28:C1:DC:B2:50        1 2023-07-28 09:13:06 2023-07-28 11:55:12  9726
     7 48:5B:39:F9:7A:48        1 2023-07-28 09:13:06 2023-07-28 11:55:11  9725
     8 E8:28:C1:DC:B2:51        1 2023-07-28 09:13:06 2023-07-28 11:55:11  9725
     9 E8:28:C1:DC:FF:F2        1 2023-07-28 09:13:06 2023-07-28 11:55:10  9724
    10 8E:55:4A:85:5B:01        1 2023-07-28 09:13:06 2023-07-28 11:55:09  9723
    # ℹ 157 more rows

5\. Обнаружить топ-10 самых быстрых точек доступа.

``` r
td_clean |> filter(!is.na(speed)) |> arrange(desc(speed)) |> 
select(BSSID, speed) |> head(10)
```

    # A tibble: 10 × 2
       BSSID             speed
       <chr>             <int>
     1 26:20:53:0C:98:E8   866
     2 96:FF:FC:91:EF:64   866
     3 CE:48:E7:86:4E:33   866
     4 8E:1F:94:96:DA:FD   866
     5 9A:75:A8:B9:04:1E   360
     6 4A:EC:1E:DB:BF:95   360
     7 56:C5:2B:9F:84:90   360
     8 E8:28:C1:DC:B2:41   360
     9 E8:28:C1:DC:B2:40   360
    10 E8:28:C1:DC:B2:42   360

6\. Отсортировать точки доступа по частоте отправки запросов (beacons) в
единицу времени по их убыванию.

``` r
td_clean |> mutate(dlit = as.numeric(last_seen - first_seen),
    beac_sec = ifelse(dlit!= 0, beacons/dlit, NA)) |> 
  arrange(desc(beac_sec)) |>
  select(BSSID, beac_sec, first_seen, last_seen, beacons, dlit)
```

    # A tibble: 167 × 6
       BSSID          beac_sec first_seen          last_seen           beacons  dlit
       <chr>             <dbl> <dttm>              <dttm>                <int> <dbl>
     1 F2:30:AB:E9:0…    0.857 2023-07-28 10:27:02 2023-07-28 10:27:09       6     7
     2 B2:CF:C0:00:4…    0.8   2023-07-28 10:40:54 2023-07-28 10:40:59       4     5
     3 3A:DA:00:F9:0…    0.556 2023-07-28 10:27:01 2023-07-28 10:27:10       5     9
     4 02:BC:15:7E:D…    0.5   2023-07-28 09:24:46 2023-07-28 09:24:48       1     2
     5 00:3E:1A:5D:1…    0.5   2023-07-28 10:34:03 2023-07-28 10:34:05       1     2
     6 76:C5:A0:70:0…    0.5   2023-07-28 11:16:36 2023-07-28 11:16:38       1     2
     7 D2:25:91:F6:6…    0.385 2023-07-28 09:45:29 2023-07-28 09:45:42       5    13
     8 BE:F1:71:D6:1…    0.174 2023-07-28 09:13:03 2023-07-28 11:50:44    1647  9461
     9 00:03:7A:1A:0…    0.167 2023-07-28 10:29:13 2023-07-28 10:29:19       1     6
    10 38:1A:52:0D:8…    0.163 2023-07-28 09:13:03 2023-07-28 10:25:02     704  4319
    # ℹ 157 more rows

#### 3. Данные клиентов

1\. Определить производителя для каждого обнаруженного устройства

``` r
#https://standards-oui.ieee.org/oui/oui.csv
oui_db <- read.csv("oui.csv")
get_mac_base <- function(mac) {
  oui <- gsub(":", "", substr(mac, 1, 8))
  result <- oui_db[oui_db$Assignment == oui, ]
  if (nrow(result) > 0) {
    return(result$Organization.Name[1])
  } else {
    return(NA)}}
```

``` r
client_clean <- client_clean |>
  mutate(manufacturer = sapply(station_mac, get_mac_base))
```

``` r
client_clean |> select(station_mac, manufacturer) |> head(10)
```

    # A tibble: 10 × 2
       station_mac       manufacturer                        
       <chr>             <chr>                               
     1 CA:66:3B:8F:56:DD <NA>                                
     2 96:35:2D:3D:85:E6 <NA>                                
     3 5C:3A:45:9E:1A:7B CHONGQING FUGUI ELECTRONICS CO.,LTD.
     4 C0:E4:34:D8:E7:E5 AzureWave Technology Inc.           
     5 5E:8E:A6:5E:34:81 <NA>                                
     6 10:51:07:CB:33:E7 Intel Corporate                     
     7 68:54:5A:40:35:9E Intel Corporate                     
     8 74:4C:A1:70:CE:F7 Liteon Technology Corporation       
     9 8A:A3:5A:33:76:57 <NA>                                
    10 CA:54:C4:8B:B5:3A <NA>                                

2\. Обнаружить устройства, которые НЕ рандомизируют свой MAC адрес

``` r
client_clean |> 
  filter(!substr(station_mac, 2, 2) %in% c("2", "6", "a", "A", "e", "E")) |>
  distinct(station_mac)
```

    # A tibble: 220 × 1
       station_mac      
       <chr>            
     1 5C:3A:45:9E:1A:7B
     2 C0:E4:34:D8:E7:E5
     3 10:51:07:CB:33:E7
     4 68:54:5A:40:35:9E
     5 74:4C:A1:70:CE:F7
     6 BC:F1:71:D4:DB:04
     7 4C:44:5B:14:76:E3
     8 A0:E7:0B:AE:D5:44
     9 00:95:69:E7:7F:35
    10 00:95:69:E7:7C:ED
    # ℹ 210 more rows

3\. Кластеризовать запросы от устройств к точкам доступа по их именам.
Определить время появления устройства в зоне радиовидимости и время
выхода его из нее.

``` r
requests <- client_clean |>
  filter(!is.na(probed_ESSIDs)) |>
  mutate(cluster = str_split(probed_ESSIDs, ",")) |>
  unnest(cluster)
  
  
clust <- requests|> group_by(station_mac, cluster) |>
  summarise(
    first_appearance = min(first_seen, na.rm = TRUE),
    last_appearance  = max(last_seen,  na.rm = TRUE),
    dlit = as.numeric(difftime(last_appearance, first_appearance)),
    .groups = "drop"
  ) |>
  arrange(station_mac, first_appearance)
```

4\. Оценить стабильность уровня сигнала внури кластера во времени.
Выявить наиболее стабильный кластер

``` r
stability <- requests |>
  group_by(cluster) |>
  summarise(
    n_requests = n(),
    mean_power = mean(power, na.rm = TRUE),
    sd_power   = if (n() == 1) 0 else sd(power, na.rm = TRUE),
    .groups = "drop") |>
  filter(n_requests >= 5) |> arrange(sd_power)  

stability |> slice_min(sd_power, n = 1)
```

    # A tibble: 1 × 4
      cluster n_requests mean_power sd_power
      <chr>        <int>      <dbl>    <dbl>
    1 MT_FREE          7      -64.7     2.69

## Вывод

В результате выполнения работы были получены знания о методах
исследования радиоэлектронной обстановки и закреплены практические
навыки использования языка программирования R для обработки данных
