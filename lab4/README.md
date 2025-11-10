# Исследование метаданных DNS трафика
nastya5908@yandex.ru

## Цель работы

1.  Закрепить практические навыки использования языка программирования R
    для обработки данных
2.  Закрепить знания основных функций обработки данных экосистемы
    tidyverse языка R
3.  Закрепить навыки исследования метаданных DNS трафика

## Исходные данные

1.  Программное обеспечение Windows 11
2.  Rstudio Desktop
3.  Интерпретатор языка R 4.5.1

## План:

1.  Импортировать и подготовить данные

2.  Проанализировать метаданные DNS трафика

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

    ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ✔ forcats   1.0.1     ✔ readr     2.1.5
    ✔ ggplot2   4.0.0     ✔ stringr   1.5.2
    ✔ lubridate 1.9.4     ✔ tibble    3.3.0
    ✔ purrr     1.1.0     ✔ tidyr     1.3.1
    ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ✖ dplyr::filter() masks stats::filter()
    ✖ dplyr::lag()    masks stats::lag()
    ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

``` r
library(readr)
```

``` r
library(httr)
```

    Warning: пакет 'httr' был собран под R версии 4.5.2

#### Подготовка данных

1\. Импортируем данные DNS

``` r
download.file("https://storage.yandexcloud.net/dataset.ctfsec/dns.zip", "dns.zip")
```

``` r
unzip("dns.zip")
```

``` r
dns <- read_tsv("dns.log", comment = "#", col_names = FALSE)
```

    Rows: 427935 Columns: 23
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: "\t"
    chr (13): X2, X3, X5, X7, X9, X10, X11, X12, X13, X14, X15, X21, X22
    dbl  (5): X1, X4, X6, X8, X20
    lgl  (5): X16, X17, X18, X19, X23

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

2\. Добавим пропущенные данные о структуре данных (назначении столбцов)

``` r
col_names <- c(
  "timestamp", "uid", "source_ip", "source_port", "destination_ip", 
  "destination_port", "protocol", "transaction_id", "query", "qclass", 
  "qclass_name", "qtype", "qtype_name", "rcode", "rcode_name", 
  "AA", "TC", "RD", "RA", "Z", "answer", "TTLS", "rejected"
)
```

``` r
colnames(dns) <- col_names
```

3\. Преобразуем данные в столбцах в нужный формат

``` r
dns <- dns |> mutate(timestamp = as_datetime(timestamp),
                      source_port = as.numeric(source_port),
                      qclass = as.numeric(qclass),
                      qtype = as.numeric(qtype))
```

    Warning: There were 2 warnings in `mutate()`.
    The first warning was:
    ℹ In argument: `qclass = as.numeric(qclass)`.
    Caused by warning:
    ! в результате преобразования созданы NA
    ℹ Run `dplyr::last_dplyr_warnings()` to see the 1 remaining warning.

Посмотрим общую структуру данных

``` r
glimpse(dns)
```

    Rows: 427,935
    Columns: 23
    $ timestamp        <dttm> 2012-03-16 12:30:05, 2012-03-16 12:30:15, 2012-03-16…
    $ uid              <chr> "CWGtK431H9XuaTN4fi", "C36a282Jljz7BsbGH", "C36a282Jl…
    $ source_ip        <chr> "192.168.202.100", "192.168.202.76", "192.168.202.76"…
    $ source_port      <dbl> 45658, 137, 137, 137, 137, 137, 137, 137, 137, 137, 1…
    $ destination_ip   <chr> "192.168.27.203", "192.168.202.255", "192.168.202.255…
    $ destination_port <dbl> 137, 137, 137, 137, 137, 137, 137, 137, 137, 137, 137…
    $ protocol         <chr> "udp", "udp", "udp", "udp", "udp", "udp", "udp", "udp…
    $ transaction_id   <dbl> 33008, 57402, 57402, 57402, 57398, 57398, 57398, 6218…
    $ query            <chr> "*\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\…
    $ qclass           <dbl> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1,…
    $ qclass_name      <chr> "C_INTERNET", "C_INTERNET", "C_INTERNET", "C_INTERNET…
    $ qtype            <dbl> 33, 32, 32, 32, 32, 32, 32, 32, 32, 32, 33, 33, 33, 1…
    $ qtype_name       <chr> "SRV", "NB", "NB", "NB", "NB", "NB", "NB", "NB", "NB"…
    $ rcode            <chr> "0", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-"…
    $ rcode_name       <chr> "NOERROR", "-", "-", "-", "-", "-", "-", "-", "-", "-…
    $ AA               <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALS…
    $ TC               <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALS…
    $ RD               <lgl> FALSE, TRUE, TRUE, TRUE, TRUE, TRUE, TRUE, TRUE, TRUE…
    $ RA               <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALS…
    $ Z                <dbl> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 1, 1, 1,…
    $ answer           <chr> "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-"…
    $ TTLS             <chr> "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-"…
    $ rejected         <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALS…

#### Анализ

4\. Сколько участников информационного обмена в сети Доброй Организации?

``` r
length(unique(c(dns$source_ip, dns$destination_ip)))
```

    [1] 1359

5\. Какое соотношение участников обмена внутри сети и участников
обращений к внешним ресурсам?

``` r
un_ip <- unique(c(dns$source_ip, dns$destination_ip))
int_ip <- un_ip[grepl("^(10\\.|192\\.168\\.|172\\.(1[6-9]|2[0-9]|3[0-1])\\.)", un_ip)]
ext_ip <- un_ip[!grepl("^(10\\.|192\\.168\\.|172\\.(1[6-9]|2[0-9]|3[0-1])\\.)", un_ip)]
length(int_ip)/length(ext_ip)
```

    [1] 13.77174

6\. Найдите топ-10 участников сети, проявляющих наибольшую сетевую
активность.

``` r
dns |> count(source_ip, sort = TRUE) |> head(10)
```

    # A tibble: 10 × 2
       source_ip           n
       <chr>           <int>
     1 10.10.117.210   75943
     2 192.168.202.93  26522
     3 192.168.202.103 18121
     4 192.168.202.76  16978
     5 192.168.202.97  16176
     6 192.168.202.141 14967
     7 10.10.117.209   14222
     8 192.168.202.110 13372
     9 192.168.203.63  12148
    10 192.168.202.106 10784

7\. Найдите топ-10 доменов, к которым обращаются пользователи сети и
соответственное количество обращений

``` r
dns |> count(query, sort = TRUE) |> head(10)
```

    # A tibble: 10 × 2
       query                                                                       n
       <chr>                                                                   <int>
     1 "teredo.ipv6.microsoft.com"                                             39273
     2 "tools.google.com"                                                      14057
     3 "www.apple.com"                                                         13390
     4 "time.apple.com"                                                        13109
     5 "safebrowsing.clients.google.com"                                       11658
     6 "*\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x… 10401
     7 "WPAD"                                                                   9134
     8 "44.206.168.192.in-addr.arpa"                                            7248
     9 "HPE8AA67"                                                               6929
    10 "ISATAP"                                                                 6569

8\. Опеределите базовые статистические характеристики (функция summary()
) интервала времени между последовательными обращениями к топ-10
доменам.

``` r
top_10_dom <- dns |> count(query, sort = TRUE) |> head(10) |> pull(query)
dns |> filter(query %in% top_10_dom) |>
     arrange(timestamp) |> group_by(query) |>
     mutate(time = lead(timestamp) - timestamp) |>
     filter(!is.na(time))|>
     summarise(
         min = min(time),
         Q1 = quantile(time, 0.25),
         median = median(time),
         Q3 = quantile(time, 0.75),
         max = max(time),
         mean = mean(time)
     )
```

    # A tibble: 10 × 7
       query                                    min   Q1    median Q3    max   mean 
       <chr>                                    <drt> <drt> <drtn> <drt> <drt> <drt>
     1 "*\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\… 0 se… 0.14… 0.500…  1.5… 5272… 11.2…
     2 "44.206.168.192.in-addr.arpa"            0 se… 2.08… 4.000… 20.0… 4967… 16.0…
     3 "HPE8AA67"                               0 se… 0.75… 0.750… 25.4… 5004… 16.6…
     4 "ISATAP"                                 0 se… 0.75… 0.759…  1.0… 5199… 17.4…
     5 "WPAD"                                   0 se… 0.75… 0.750…  1.1… 5004… 12.6…
     6 "safebrowsing.clients.google.com"        0 se… 0.00… 1.000…  2.0… 4995… 10.0…
     7 "teredo.ipv6.microsoft.com"              0 se… 0.00… 0.000…  0.5… 5038…  2.9…
     8 "time.apple.com"                         0 se… 0.36… 1.760…  4.7… 5092…  8.6…
     9 "tools.google.com"                       0 se… 0.00… 0.000…  1.0… 5036…  8.1…
    10 "www.apple.com"                          0 se… 0.00… 1.000…  3.0… 5096…  8.6…

9\. Часто вредоносное программное обеспечение использует DNS канал в
качестве канала управления, периодически отправляя запросы на
подконтрольный злоумышленникам DNS сервер. По периодическим запросам на
один и тот же домен можно выявить скрытый DNS канал. Есть ли такие IP
адреса в исследуемом датасете?

``` r
dns |>
  arrange(source_ip, query, timestamp) |>
  group_by(source_ip, query) |>
  mutate(time = as.numeric(lead(timestamp) - timestamp)) |>
  filter(!is.na(time)) |>
  summarise(requests = n() + 1,
    avg_interval = mean(time)) |>
  filter(requests >= 10, avg_interval <= 30) |>
  group_by(source_ip) |>
  summarise(
    pd_domains = n(),
    total_requests = sum(requests),
  ) |>
  arrange(desc(pd_domains))
```

    `summarise()` has grouped output by 'source_ip'. You can override using the
    `.groups` argument.

    # A tibble: 106 × 3
       source_ip       pd_domains total_requests
       <chr>                <int>          <dbl>
     1 192.168.202.97         281           6614
     2 192.168.202.71          51           1203
     3 192.168.202.84          48           1157
     4 10.10.117.210           47          69031
     5 192.168.202.79          44           1141
     6 192.168.202.100         37            578
     7 192.168.202.103         28           9988
     8 192.168.202.110         23           2378
     9 192.168.202.106         18           1967
    10 192.168.204.45          16            363
    # ℹ 96 more rows

#### Обогащение данных

10\. Определите местоположение (страну, город) и организацию-провайдера
для топ-10 доменов. Для этого можно использовать сторонние сервисы,
например http://ip-api.com (API-эндпоинт – http://ip-api.com/json).

``` r
top_10_dom <- dns |> count(query, sort = TRUE) |> head(10) |> pull(query)

get_geo <- function(query) {
  tryCatch({
    response <- GET(paste0("http://ip-api.com/json/", query))
    data <- content(response, "parsed")
    return(list(
      query=query,
      ip = data$query %||% NA,
      country = data$country %||% NA,
      city = data$city %||% NA, 
      isp = data$isp %||% NA
    ))
  }, error = function(e) {
    return(list(ip = NA, country = NA, city = NA, isp = NA))
  })
}
results <- map_dfr(top_10_dom, function(query) {
  geo <- get_geo(query)
})

print(results)
```

    # A tibble: 10 × 5
       query                                               ip    country city  isp  
       <chr>                                               <chr> <chr>   <chr> <chr>
     1 "teredo.ipv6.microsoft.com"                         "ter… <NA>    <NA>  <NA> 
     2 "tools.google.com"                                  "142… United… Moun… Goog…
     3 "www.apple.com"                                     "2.1… United… Lond… Akam…
     4 "time.apple.com"                                    "17.… United… Slou… Appl…
     5 "safebrowsing.clients.google.com"                   "142… United… Moun… Goog…
     6 "*\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x… "*\\… <NA>    <NA>  <NA> 
     7 "WPAD"                                              "WPA… <NA>    <NA>  <NA> 
     8 "44.206.168.192.in-addr.arpa"                       "44.… <NA>    <NA>  <NA> 
     9 "HPE8AA67"                                          "HPE… <NA>    <NA>  <NA> 
    10 "ISATAP"                                            "ISA… <NA>    <NA>  <NA> 

## Вывод

В результате выполнения работы были закреплены знания основных функций
обработки данных экосистемы tidyverse и получены навыки исследования
метаданных DNS трафика
