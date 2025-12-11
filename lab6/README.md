# Исследование вредоносной активности в домене Windows
nastya5908@yandex.ru

## Цель работы

1.  Закрепить навыки исследования данных журнала Windows Active
    Directory
2.  Изучить структуру журнала системы Windows Active Directory
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
library(tidyr)
```

#### Подготовка данных

1\. Импортируйте данные в R. Это можно выполнить с помощью
jsonlite::stream_in(file()). Датасет находится по адресу
https://storage.yandexcloud.net/iamcth-data/dataset.tar.gz.

``` r
download.file(
  url = "https://storage.yandexcloud.net/iamcth-data/dataset.tar.gz",
  destfile = "dataset.tar.gz"
)
```

``` r
untar("dataset.tar.gz", exdir = ".")
data <-jsonlite::stream_in(file("caldera_attack_evals_round1_day1_2019-10-20201108.json"), verbose = FALSE)
```

2\. Привести датасеты в вид “аккуратных данных”, преобразовать типы
столбцов в соответствии с типом данных

``` r
data <- data |>
  rename(metadata = `@metadata`, 
         timestamp = `@timestamp`)
```

3\. Просмотрите общую структуру данных с помощью функции glimpse()

``` r
glimpse(data)
```

    Rows: 101,904
    Columns: 9
    $ timestamp <chr> "2019-10-20T20:11:06.937Z", "2019-10-20T20:11:07.101Z", "201…
    $ metadata  <df[,4]> <data.frame[26 x 4]>
    $ event     <df[,4]> <data.frame[26 x 4]>
    $ log       <df[,1]> <data.frame[26 x 1]>
    $ message   <chr> "A token right was adjusted.\n\nSubject:\n\tSecurity ID:\…
    $ winlog    <df[,16]> <data.frame[26 x 16]>
    $ ecs       <df[,1]> <data.frame[26 x 1]>
    $ host      <df[,1]> <data.frame[26 x 1]>
    $ agent     <df[,5]> <data.frame[26 x 5]>

#### Анализ

1\. Раскройте датафрейм избавившись от вложенных датафреймов.

``` r
data <- unnest(data, cols = c(metadata, event, log, winlog, ecs, host, agent), 
                      names_sep = "_")
```

``` r
glimpse(data)
```

    Rows: 101,904
    Columns: 34
    $ timestamp            <chr> "2019-10-20T20:11:06.937Z", "2019-10-20T20:11:07.…
    $ metadata_beat        <chr> "winlogbeat", "winlogbeat", "winlogbeat", "winlog…
    $ metadata_type        <chr> "_doc", "_doc", "_doc", "_doc", "_doc", "_doc", "…
    $ metadata_version     <chr> "7.4.0", "7.4.0", "7.4.0", "7.4.0", "7.4.0", "7.4…
    $ metadata_topic       <chr> "winlogbeat", "winlogbeat", "winlogbeat", "winlog…
    $ event_created        <chr> "2019-10-20T20:11:09.988Z", "2019-10-20T20:11:09.…
    $ event_kind           <chr> "event", "event", "event", "event", "event", "eve…
    $ event_code           <int> 4703, 4673, 10, 10, 10, 10, 11, 10, 10, 10, 10, 7…
    $ event_action         <chr> "Token Right Adjusted Events", "Sensitive Privile…
    $ log_level            <chr> "information", "information", "information", "inf…
    $ message              <chr> "A token right was adjusted.\n\nSubject:\n\tSecur…
    $ winlog_event_data    <df[,234]> <data.frame[26 x 234]>
    $ winlog_event_id      <int> 4703, 4673, 10, 10, 10, 10, 11, 10, 10, 10, …
    $ winlog_provider_name <chr> "Microsoft-Windows-Security-Auditing", "Microsoft…
    $ winlog_api           <chr> "wineventlog", "wineventlog", "wineventlog", "win…
    $ winlog_record_id     <int> 50588, 104875, 226649, 153525, 163488, 153526, 13…
    $ winlog_computer_name <chr> "HR001.shire.com", "HFDC01.shire.com", "IT001.shi…
    $ winlog_process       <df[,2]> <data.frame[26 x 2]>
    $ winlog_keywords      <list> "Audit Success", "Audit Failure", <NULL>, <NULL>,…
    $ winlog_provider_guid <chr> "{54849625-5478-4994-a5ba-3e3b0328c30d}", "{54…
    $ winlog_channel       <chr> "security", "Security", "Microsoft-Windows-Sysmo…
    $ winlog_task          <chr> "Token Right Adjusted Events", "Sensitive Privile…
    $ winlog_opcode        <chr> "Info", "Info", "Info", "Info", "Info", "Info", "…
    $ winlog_version       <int> NA, NA, 3, 3, 3, 3, 2, 3, 3, 3, 3, 3, 3, 3, NA, 3…
    $ winlog_user          <df[,4]> <data.frame[26 x 4]>
    $ winlog_activity_id   <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog_user_data     <df[,30]> <data.frame[26 x 30]>
    $ ecs_version          <chr> "1.1.0", "1.1.0", "1.1.0", "1.1.0", "1.1.0", "1.1…
    $ host_name            <chr> "WECServer", "WECServer", "WECServer", "WECSer…
    $ agent_ephemeral_id   <chr> "b372be1f-ba0a-4d7e-b4df-79eac86e1fde", "b372be1f…
    $ agent_hostname       <chr> "WECServer", "WECServer", "WECServer", "WECSe…
    $ agent_id             <chr> "d347d9a4-bff4-476c-b5a4-d51119f78250", "d347d9a4…
    $ agent_version        <chr> "7.4.0", "7.4.0", "7.4.0", "7.4.0", "7.4.0", "7.4…
    $ agent_type           <chr> "winlogbeat", "winlogbeat", "winlogbeat", "winlog…

2\. Минимизируйте количество колонок в датафрейме – уберите колонки с
единственным значением параметра.

``` r
data <- data |> 
  select(where(~ n_distinct(., na.rm = TRUE) > 1))
glimpse(data)
```

    Rows: 101,904
    Columns: 19
    $ timestamp            <chr> "2019-10-20T20:11:06.937Z", "2019-10-20T20:11:07.…
    $ event_created        <chr> "2019-10-20T20:11:09.988Z", "2019-10-20T20:11:09.…
    $ event_code           <int> 4703, 4673, 10, 10, 10, 10, 11, 10, 10, 10, 10, 7…
    $ event_action         <chr> "Token Right Adjusted Events", "Sensitive Privile…
    $ log_level            <chr> "information", "information", "information", "inf…
    $ message              <chr> "A token right was adjusted.\n\nSubject:\n\tSecur…
    $ winlog_event_id      <int> 4703, 4673, 10, 10, 10, 10, 11, 10, 10, 10, 10, 7…
    $ winlog_provider_name <chr> "Microsoft-Windows-Security-Auditing", "Microsoft…
    $ winlog_record_id     <int> 50588, 104875, 226649, 153525, 163488, 153526, 13…
    $ winlog_computer_name <chr> "HR001.shire.com", "HFDC01.shire.com", "IT001.shi…
    $ winlog_process       <df[,2]> <data.frame[26 x 2]>
    $ winlog_keywords      <list> "Audit Success", "Audit Failure", <NULL>, <NUL…
    $ winlog_provider_guid <chr> "{54849625-5478-4994-a5ba-3e3b0328c30d}", "{5484…
    $ winlog_channel       <chr> "security", "Security", "Microsoft-Windows-Sysmon…
    $ winlog_task          <chr> "Token Right Adjusted Events", "Sensitive Privile…
    $ winlog_opcode        <chr> "Info", "Info", "Info", "Info", "Info", "Info", "…
    $ winlog_version       <int> NA, NA, 3, 3, 3, 3, 2, 3, 3, 3, 3, 3, 3, 3, NA, 3…
    $ winlog_user          <df[,4]> <data.frame[26 x 4]>
    $ winlog_activity_id   <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…

3\. Какое количество хостов представлено в данном датасете?

``` r
n_distinct(data$winlog_computer_name)
```

    [1] 5

4\. Подготовьте датафрейм с расшифровкой Windows Event_ID, приведите
типы данных к типу их значений.

``` r
library(xml2)
library(rvest)
```


    Присоединяю пакет: 'rvest'

    Следующий объект скрыт от 'package:readr':

        guess_encoding

``` r
webpage_url <- "https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/plan/appendix-l--events-to-monitor"
webpage <- xml2::read_html(webpage_url)
event_df <- rvest::html_table(webpage, fill = TRUE)[[1]]
```

``` r
event_df <- event_df |>
  rename(
    event_id = `Current Windows Event ID`,
    legacy_event_id = `Legacy Windows Event ID`,
    criticality = `Potential Criticality`,
    description = `Event Summary`
  ) |>
  mutate(
    event_id = as.integer(event_id),
    legacy_event_id = as.integer(legacy_event_id),
  )
```

    Warning: There were 2 warnings in `mutate()`.
    The first warning was:
    ℹ In argument: `event_id = as.integer(event_id)`.
    Caused by warning:
    ! в результате преобразования созданы NA
    ℹ Run `dplyr::last_dplyr_warnings()` to see the 1 remaining warning.

``` r
glimpse(event_df)
```

    Rows: 381
    Columns: 4
    $ event_id        <int> 4618, 4649, 4719, 4765, 4766, 4794, 4897, 4964, 5124, …
    $ legacy_event_id <int> NA, NA, 612, NA, NA, NA, 801, NA, NA, 550, 517, NA, NA…
    $ criticality     <chr> "High", "High", "High", "High", "High", "High", "High"…
    $ description     <chr> "A monitored security event pattern has occurred.", "A…

5\. Есть ли в логе события с высоким и средним уровнем значимости?
Сколько их?

``` r
result <- data |>
  select(event_id = winlog_event_id) |>
  inner_join(event_df |> select(event_id, criticality), by = "event_id") |>
  filter(criticality %in% c("High", "Medium")) |>
  count(criticality, .drop = FALSE)

print(result)
```

    # A tibble: 0 × 2
    # ℹ 2 variables: criticality <chr>, n <int>

``` r
cat(sum(result$n), "событий")
```

    0 событий

## Вывод

В результате выполнения работы были получены навыки исследования данных
журнала Windows Active Directory и закреплены практические навыки
использования языка программирования R для обработки данных
