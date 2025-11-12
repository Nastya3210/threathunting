# Использование технологии Yandex Query для анализа данных сетевой активности
nastya5908@yandex.ru

## Цель работы

1.  Изучить возможности технологии Yandex Query для анализа
    структурированных наборов данных
2.  Получить навыки построения аналитического пайплайна для анализа
    данных с помощью сервисов Yandex Cloud
3.  Закрепить практические навыки использования SQL для анализа данных
    сетевой активности в сегментированной корпоративной сети

## Исходные данные

1.  Файл yaqry_dataset.pqt в бакете arrow-datasets S3 хранилища Yandex
    Object Storage
2.  Облачное решение для анализа данных Yandex Query

## План:

1.  Проверить доступность данных в Yandex Object Storage
2.  Подключить бакет как источник данных для Yandex Query
3.  Проанализировать данные и выполнить задания

## Шаги

``` r
sessionInfo()
```

    R version 4.5.1 (2025-06-13 ucrt)
    Platform: x86_64-w64-mingw32/x64
    Running under: Windows 11 x64 (build 26200)

    Matrix products: default
      LAPACK version 3.12.1

    locale:
    [1] LC_COLLATE=Russian_Russia.utf8  LC_CTYPE=Russian_Russia.utf8   
    [3] LC_MONETARY=Russian_Russia.utf8 LC_NUMERIC=C                   
    [5] LC_TIME=Russian_Russia.utf8    

    time zone: Europe/Moscow
    tzcode source: internal

    attached base packages:
    [1] stats     graphics  grDevices utils     datasets  methods   base     

    loaded via a namespace (and not attached):
     [1] compiler_4.5.1    fastmap_1.2.0     cli_3.6.5         tools_4.5.1      
     [5] htmltools_0.5.8.1 rstudioapi_0.17.1 yaml_2.3.10       rmarkdown_2.29   
     [9] knitr_1.50        jsonlite_2.0.0    xfun_0.53         digest_0.6.37    
    [13] rlang_1.1.6       evaluate_1.0.5   

1.  Проверим доступность данных (файл yaqry_dataset.pqt) в бакете
    arrow-datasets S3 хранилища Yandex Object Storage ![](img/1.png)
    Данные доступны

#### Подключим бакет как источник данных для Yandex Query

1.  Создадим соединение для бакета S3 в хранилище ![](img/2.png)
2.  Сделаем привязку данных ![](img/3.png)
3.  Настроим привязку данных ![](img/4.png)
4.  Проверим привязку данных - сделаем аналитический запрос для вывода
    первых 100 строк

``` sql
SELECT * FROM `yaqry_dataset`
LIMIT 100;
```

![](img/5.png)

#### Решим задания:

1.  Известно, что IP адреса внутренней сети начинаются с октетов,
    принадлежащих интервалу 12-14. Определите количество хостов
    внутренней сети, представленных в датасете.

``` sql
SELECT COUNT(distinct ip)
FROM (
    SELECT src AS ip FROM `yaqry_dataset` 
    WHERE src LIKE '12.%' OR src LIKE '13.%' OR src LIKE '14.%'
    UNION
    SELECT dst AS ip FROM `yaqry_dataset`
    WHERE dst LIKE '12.%' OR dst LIKE '13.%' OR dst LIKE '14.%')
```

![](img/6.png) 2. Определите суммарный объем исходящего трафика

``` sql
SELECT SUM(bytes)
FROM `yaqry_dataset`
WHERE (src LIKE '12.%' OR src LIKE '13.%' OR src LIKE '14.%') AND
(dst NOT LIKE '12.%' AND dst NOT LIKE '13.%' AND dst NOT LIKE '14.%')
```

![](img/7.png) 3. Определите суммарный объем входящего трафика

``` sql
SELECT SUM(bytes)
FROM `yaqry_dataset`
WHERE (dst LIKE '12.%' OR dst LIKE '13.%' OR dst LIKE '14.%') AND
(src NOT LIKE '12.%' AND src NOT LIKE '13.%' AND src NOT LIKE '14.%')
```

![](img/8.png)

## Вывод

При выполнении работы были изучены технологии Yandex Query для анализа
структурированных наборов данных, получены навыки для построения
аналитического пайплайна для анализа данных с помощью сервисов Yandex
Cloud и закреплены практические навыки использования SQL для анализа
данных сетевой активности в сегментированной корпоративной сети
