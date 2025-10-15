# Основы обработки данных с помощью R и Dplyr
nastya5908@yandex.ru

## Цель работы

1.  Развить практические навыки использования языка программирования R
    для обработки данных
2.  Закрепить знания базовых типов данных языка R
3.  Развить практические навыки использования функций обработки данных
    пакета `dplyr` – функции
    `select(), filter(), mutate(), arrange(), group_by()`

## Исходные данные

1.  Программное обеспечение Windows 11
2.  Rstudio Desktop
3.  Интерпретатор языка R 4.5.1

## План:

1.  Загрузить пакет dplyr

2.  Проанализировать встроенный в пакет dplyr набор данных starwars с
    помощью языка R и ответить на вопросы

3.  Составить отчет и выложить его и исходный qmd/rmd файл в свой
    репозиторий

## Шаги

1\. Установим и подключим пакет dplyr

``` r
library(dplyr)
```


    Присоединяю пакет: 'dplyr'

    Следующие объекты скрыты от 'package:stats':

        filter, lag

    Следующие объекты скрыты от 'package:base':

        intersect, setdiff, setequal, union

2\. Проанализируем набор starwars

Сколько строк в датафрейме?

``` r
starwars |> nrow()
```

    [1] 87

Сколько столбцов в датафрейме?

``` r
starwars |> ncol()
```

    [1] 14

Как просмотреть примерный вид датафрейма?

``` r
starwars |> glimpse()
```

    Rows: 87
    Columns: 14
    $ name       <chr> "Luke Skywalker", "C-3PO", "R2-D2", "Darth Vader", "Leia Or…
    $ height     <int> 172, 167, 96, 202, 150, 178, 165, 97, 183, 182, 188, 180, 2…
    $ mass       <dbl> 77.0, 75.0, 32.0, 136.0, 49.0, 120.0, 75.0, 32.0, 84.0, 77.…
    $ hair_color <chr> "blond", NA, NA, "none", "brown", "brown, grey", "brown", N…
    $ skin_color <chr> "fair", "gold", "white, blue", "white", "light", "light", "…
    $ eye_color  <chr> "blue", "yellow", "red", "yellow", "brown", "blue", "blue",…
    $ birth_year <dbl> 19.0, 112.0, 33.0, 41.9, 19.0, 52.0, 47.0, NA, 24.0, 57.0, …
    $ sex        <chr> "male", "none", "none", "male", "female", "male", "female",…
    $ gender     <chr> "masculine", "masculine", "masculine", "masculine", "femini…
    $ homeworld  <chr> "Tatooine", "Tatooine", "Naboo", "Tatooine", "Alderaan", "T…
    $ species    <chr> "Human", "Droid", "Droid", "Human", "Human", "Human", "Huma…
    $ films      <list> <"A New Hope", "The Empire Strikes Back", "Return of the J…
    $ vehicles   <list> <"Snowspeeder", "Imperial Speeder Bike">, <>, <>, <>, "Imp…
    $ starships  <list> <"X-wing", "Imperial shuttle">, <>, <>, "TIE Advanced x1",…

Сколько уникальных рас персонажей (species) представлено в данных?

``` r
starwars |> select(species) |> filter(!is.na(species)) |> unique() |> count() 
```

    # A tibble: 1 × 1
          n
      <int>
    1    37

Найти самого высокого персонажа

``` r
starwars |> select(name, height) |> arrange(desc(height)) |> head(1) |> as.data.frame()
```

             name height
    1 Yarael Poof    264

Найти всех персонажей ниже 170

``` r
starwars |> select(name, height) |> filter(height < 170) |> as.data.frame()
```

                        name height
    1                  C-3PO    167
    2                  R2-D2     96
    3            Leia Organa    150
    4     Beru Whitesun Lars    165
    5                  R5-D4     97
    6                   Yoda     66
    7             Mon Mothma    150
    8  Wicket Systri Warrick     88
    9              Nien Nunb    160
    10                 Watto    137
    11               Sebulba    112
    12        Shmi Skywalker    163
    13          Ratts Tyerel     79
    14              Dud Bolt     94
    15               Gasgano    122
    16        Ben Quadinaros    163
    17                 Cordé    157
    18         Barriss Offee    166
    19                 Dormé    165
    20            Zam Wesell    168
    21            Jocasta Nu    167
    22                R4-P17     96

Подсчитать ИМТ (индекс массы тела) для всех персонажей

``` r
starwars |> mutate(imt = round(mass/(height^2),5)) |> select(name, imt) |> as.data.frame()
```

                        name     imt
    1         Luke Skywalker 0.00260
    2                  C-3PO 0.00269
    3                  R2-D2 0.00347
    4            Darth Vader 0.00333
    5            Leia Organa 0.00218
    6              Owen Lars 0.00379
    7     Beru Whitesun Lars 0.00275
    8                  R5-D4 0.00340
    9      Biggs Darklighter 0.00251
    10        Obi-Wan Kenobi 0.00232
    11      Anakin Skywalker 0.00238
    12        Wilhuff Tarkin      NA
    13             Chewbacca 0.00215
    14              Han Solo 0.00247
    15                Greedo 0.00247
    16 Jabba Desilijic Tiure 0.04434
    17        Wedge Antilles 0.00266
    18      Jek Tono Porkins 0.00340
    19                  Yoda 0.00390
    20             Palpatine 0.00260
    21             Boba Fett 0.00234
    22                 IG-88 0.00350
    23                 Bossk 0.00313
    24      Lando Calrissian 0.00252
    25                 Lobot 0.00258
    26                Ackbar 0.00256
    27            Mon Mothma      NA
    28          Arvel Crynyd      NA
    29 Wicket Systri Warrick 0.00258
    30             Nien Nunb 0.00266
    31          Qui-Gon Jinn 0.00239
    32           Nute Gunray 0.00247
    33         Finis Valorum      NA
    34         Padmé Amidala 0.00131
    35         Jar Jar Binks 0.00172
    36          Roos Tarpals 0.00163
    37            Rugor Nass      NA
    38              Ric Olié      NA
    39                 Watto      NA
    40               Sebulba 0.00319
    41         Quarsh Panaka      NA
    42        Shmi Skywalker      NA
    43            Darth Maul 0.00261
    44           Bib Fortuna      NA
    45           Ayla Secura 0.00174
    46          Ratts Tyerel 0.00240
    47              Dud Bolt 0.00509
    48               Gasgano      NA
    49        Ben Quadinaros 0.00245
    50            Mace Windu 0.00238
    51          Ki-Adi-Mundi 0.00209
    52             Kit Fisto 0.00226
    53             Eeth Koth      NA
    54            Adi Gallia 0.00148
    55           Saesee Tiin      NA
    56           Yarael Poof      NA
    57              Plo Koon 0.00226
    58            Mas Amedda      NA
    59          Gregar Typho 0.00248
    60                 Cordé      NA
    61           Cliegg Lars      NA
    62     Poggle the Lesser 0.00239
    63       Luminara Unduli 0.00194
    64         Barriss Offee 0.00181
    65                 Dormé      NA
    66                 Dooku 0.00215
    67   Bail Prestor Organa      NA
    68            Jango Fett 0.00236
    69            Zam Wesell 0.00195
    70       Dexter Jettster 0.00260
    71               Lama Su 0.00168
    72               Taun We      NA
    73            Jocasta Nu      NA
    74                R4-P17      NA
    75            Wat Tambor 0.00129
    76              San Hill      NA
    77              Shaak Ti 0.00180
    78              Grievous 0.00341
    79               Tarfful 0.00248
    80       Raymus Antilles 0.00224
    81             Sly Moore 0.00151
    82            Tion Medon 0.00189
    83                  Finn      NA
    84                   Rey      NA
    85           Poe Dameron      NA
    86                   BB8      NA
    87        Captain Phasma      NA

Найти 10 самых “вытянутых” персонажей. “Вытянутость” оценить по
отношению массы (mass) к росту (height) персонажей.

``` r
starwars |> mutate(vt = round(mass/height,5)) |> select(name, vt) |> arrange(vt) |> head(10) |> as.data.frame()
```

                        name      vt
    1           Ratts Tyerel 0.18987
    2  Wicket Systri Warrick 0.22727
    3          Padmé Amidala 0.24324
    4             Wat Tambor 0.24870
    5                   Yoda 0.25758
    6              Sly Moore 0.26966
    7             Adi Gallia 0.27174
    8          Barriss Offee 0.30120
    9            Ayla Secura 0.30899
    10              Shaak Ti 0.32022

Найти средний возраст персонажей каждой расы вселенной Звездных войн.

``` r
starwars |> filter(!is.na(species)) |> filter(!is.na(birth_year)) |> group_by(species) |> summarise(avg_age = mean(100 - birth_year)) |> select(species, avg_age) |> as.data.frame()
```

              species    avg_age
    1          Cerean    8.00000
    2           Droid   46.66667
    3            Ewok   92.00000
    4          Gungan   48.00000
    5           Human   46.25769
    6            Hutt -500.00000
    7         Kel Dor   78.00000
    8        Mirialan   51.00000
    9    Mon Calamari   59.00000
    10         Rodian   56.00000
    11     Trandoshan   47.00000
    12        Twi'lek   52.00000
    13        Wookiee -100.00000
    14 Yoda's species -796.00000
    15         Zabrak   46.00000

Найти самый распространенный цвет глаз персонажей вселенной Звездных
войн.

``` r
starwars |> group_by(eye_color) |> summarise(cnt = n()) |> select(eye_color, cnt) |> arrange(desc(cnt)) |> head(1) |> as.data.frame()
```

      eye_color cnt
    1     brown  21

Подсчитать среднюю длину имени в каждой расе вселенной Звездных войн.

``` r
starwars |> filter(!is.na(species)) |> group_by(species) |> summarise(avg_len = mean(nchar(name))) |> select(species, avg_len) |> as.data.frame()
```

              species   avg_len
    1          Aleena 12.000000
    2        Besalisk 15.000000
    3          Cerean 12.000000
    4        Chagrian 10.000000
    5        Clawdite 10.000000
    6           Droid  4.833333
    7             Dug  7.000000
    8            Ewok 21.000000
    9       Geonosian 17.000000
    10         Gungan 11.666667
    11          Human 11.342857
    12           Hutt 21.000000
    13       Iktotchi 11.000000
    14        Kaleesh  8.000000
    15       Kaminoan  7.000000
    16        Kel Dor  8.000000
    17       Mirialan 14.000000
    18   Mon Calamari  6.000000
    19           Muun  8.000000
    20       Nautolan  9.000000
    21      Neimodian 11.000000
    22         Pau'an 10.000000
    23       Quermian 11.000000
    24         Rodian  6.000000
    25        Skakoan 10.000000
    26      Sullustan  9.000000
    27     Tholothian 10.000000
    28        Togruta  8.000000
    29          Toong 14.000000
    30      Toydarian  5.000000
    31     Trandoshan  5.000000
    32        Twi'lek 11.000000
    33     Vulptereen  8.000000
    34        Wookiee  8.000000
    35          Xexto  7.000000
    36 Yoda's species  4.000000
    37         Zabrak  9.500000

## Вывод

В результате выполнения работы были получены практические навыки
использования функций обработки данных пакета dplyr.
