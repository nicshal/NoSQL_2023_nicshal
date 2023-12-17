# Описание/Пошаговая инструкция выполнения домашнего задания:
> - Необходимо, используя туториал https://clickhouse.tech/docs/ru/getting-started/tutorial/ развернуть БД;
> - выполнить импорт тестовой БД;
> - выполнить несколько запросов и оценить скорость выполнения.

## Описание процесса установки (локальная установка)
```sh
nicshal@nicshal-ubuntu:/etc$ sudo apt-get install -y apt-transport-https ca-certificates dirmngr
nicshal@nicshal-ubuntu:/etc$ sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 8919F6BD2B48D754

nicshal@nicshal-ubuntu:/etc$ echo "deb https://packages.clickhouse.com/deb stable main" | sudo tee  /etc/apt/sources.list.d/clickhouse.list
nicshal@nicshal-ubuntu:/etc$ sudo apt-get update

nicshal@nicshal-ubuntu:/etc$ sudo apt-get install -y clickhouse-server clickhouse-client

-- стартуем сервер
nicshal@nicshal-ubuntu:/etc$ sudo service clickhouse-server start

-- проверяем статус
nicshal@nicshal-ubuntu:/etc$ sudo service clickhouse-server status
[sudo] пароль для nicshal: 
● clickhouse-server.service - ClickHouse Server (analytic DBMS for big data)
     Loaded: loaded (/lib/systemd/system/clickhouse-server.service; enabled; vendor preset: enabled)
     Active: active (running) since Sun 2023-12-17 18:37:05 MSK; 40min ago
   Main PID: 112691 (clickhouse-serv)
      Tasks: 702 (limit: 18896)
     Memory: 1.2G
     CGroup: /system.slice/clickhouse-server.service
             ├─112690 clickhouse-watchdog        --config=/etc/clickhouse-server/config.xml --pid-file=/run/clickhouse-server/clickhouse-server.pid
             └─112691 /usr/bin/clickhouse-server --config=/etc/clickhouse-server/config.xml --pid-file=/run/clickhouse-server/clickhouse-server.pid

дек 17 18:37:04 nicshal-ubuntu clickhouse-server[112690]: Processing configuration file '/etc/clickhouse-server/config.xml'.
дек 17 18:37:04 nicshal-ubuntu clickhouse-server[112690]: Logging trace to /var/log/clickhouse-server/clickhouse-server.log
дек 17 18:37:04 nicshal-ubuntu clickhouse-server[112690]: Logging errors to /var/log/clickhouse-server/clickhouse-server.err.log
дек 17 18:37:04 nicshal-ubuntu systemd[1]: clickhouse-server.service: Supervising process 112691 which is not our child. We'll most likely not notice when it exits.
дек 17 18:37:04 nicshal-ubuntu clickhouse-server[112691]: Processing configuration file '/etc/clickhouse-server/config.xml'.
дек 17 18:37:04 nicshal-ubuntu clickhouse-server[112691]: Saved preprocessed configuration to '/var/lib/clickhouse/preprocessed_configs/config.xml'.
дек 17 18:37:04 nicshal-ubuntu clickhouse-server[112691]: Processing configuration file '/etc/clickhouse-server/users.xml'.
дек 17 18:37:04 nicshal-ubuntu clickhouse-server[112691]: Merging configuration file '/etc/clickhouse-server/users.d/default-password.xml'.
дек 17 18:37:04 nicshal-ubuntu clickhouse-server[112691]: Saved preprocessed configuration to '/var/lib/clickhouse/preprocessed_configs/users.xml'.
дек 17 18:37:05 nicshal-ubuntu systemd[1]: Started ClickHouse Server (analytic DBMS for big data).

-- заходим в CLInicshal@nicshal-ubuntu:/etc$ clickhouse-client --password
ClickHouse client version 23.11.2.11 (official build).
Password for user (default): 
Connecting to localhost:9000 as user default.
Connected to ClickHouse server version 23.11.2.

Warnings:
 * Delay accounting is not enabled, OSIOWaitMicroseconds will not be gathered. Check /proc/sys/kernel/task_delayacct

nicshal-ubuntu :) 

```

> - Далее закачал тестовые dataset'ы с https://recipenlg.cs.put.poznan.pl/dataset (набор данных кулинарных рецептов)
> - Распаковал архив в /home/nicshal/dataset/full_dataset.csv

## Описание процесса заливки данных в базу
```sh
-- создаем таблицу
nicshal-ubuntu :) CREATE TABLE recipes
(
    title String,
    ingredients Array(String),
    directions Array(String),
    link String,
    source LowCardinality(String),
    NER Array(String)
) ENGINE = MergeTree ORDER BY title;

-- добавляем данные в таблицу
nicshal@nicshal-ubuntu:/etc$ clickhouse-client --password --query "
    INSERT INTO recipes
    SELECT
        title,
        JSONExtract(ingredients, 'Array(String)'),
        JSONExtract(directions, 'Array(String)'),
        link,
        source,
        JSONExtract(NER, 'Array(String)')
    FROM input('num UInt32, title String, ingredients String, directions String, link String, source LowCardinality(String), NER String')
    FORMAT CSVWithNames
" --input_format_with_names_use_header 0 --format_csv_allow_single_quote 0 --input_format_allow_errors_num 10 < /home/nicshal/dataset/full_dataset.csv

```

### Пояснение:
> - набор данных представлен в формате CSV и требует некоторой предварительной обработки при вставке. Для предварительной обработки используется табличная функция input;
> - структура CSV-файла задается в аргументе табличной функции input;
> - поле num (номер строки) не нужно — оно считывается из файла, но игнорируется;
> - при загрузке используется FORMAT CSVWithNames, но заголовок в CSV будет проигнорирован (параметром командной строки --input_format_with_names_use_header 0), поскольку заголовок не содержит имени первого поля;
> - в файле CSV для разделения строк используются только двойные кавычки. Но некоторые строки не заключены в двойные кавычки, и чтобы одинарная кавычка не рассматривалась как заключающая, используется параметр --format_csv_allow_single_quote 0;
> - некоторые строки из CSV не могут быть считаны корректно, поскольку они начинаются с символов\M/, тогда как в CSV начинаться с обратной косой черты могут только символы \N, которые распознаются как NULL в SQL. Поэтому используется параметр --input_format_allow_errors_num 10, разрешающий пропустить до десяти некорректных записей;
> - массивы ingredients, directions и NER представлены в необычном виде: они сериализуются в строку формата JSON, а затем помещаются в CSV — тогда они могут считываться и обрабатываться как обычные строки (String). Чтобы преобразовать строку в массив, используется функция JSONExtract.


## Проверяем добавленные данные
```sh
nicshal-ubuntu :) SELECT count() FROM recipes;

SELECT count()
FROM recipes

Query id: 684867bf-16ad-4165-aedc-ffa9f48fb812

┌─count()─┐
│ 2231142 │
└─────────┘

1 row in set. Elapsed: 0.003 sec. 

```

## Примеры запросов
```sh
-- самые упоминаемые ингредиенты
nicshal-ubuntu :) SELECT
    arrayJoin(NER) AS k,
    count() AS c
FROM recipes
GROUP BY k
ORDER BY c DESC
LIMIT 50

SELECT
    arrayJoin(NER) AS k,
    count() AS c
FROM recipes
GROUP BY k
ORDER BY c DESC
LIMIT 50

Query id: ddc6f4de-e63e-4ae1-9741-5e7854b0b579

┌─k────────────────────┬──────c─┐
│ salt                 │ 890741 │
│ sugar                │ 620027 │
│ butter               │ 493823 │
│ flour                │ 466110 │
│ eggs                 │ 401276 │
│ onion                │ 372469 │
│ garlic               │ 358364 │
│ milk                 │ 346769 │
│ water                │ 326092 │
│ vanilla              │ 270381 │
│ olive oil            │ 197877 │
│ pepper               │ 179305 │
│ brown sugar          │ 174447 │
│ tomatoes             │ 163933 │
│ egg                  │ 160507 │
│ baking powder        │ 148277 │
│ lemon juice          │ 146414 │
│ Salt                 │ 122558 │
│ cinnamon             │ 117927 │
│ sour cream           │ 116682 │
│ cream cheese         │ 114423 │
│ margarine            │ 112742 │
│ celery               │ 112676 │
│ baking soda          │ 110690 │
│ parsley              │ 102151 │
│ chicken              │ 101505 │
│ onions               │  98903 │
│ vegetable oil        │  91395 │
│ oil                  │  85600 │
│ mayonnaise           │  84822 │
│ pecans               │  79741 │
│ nuts                 │  78471 │
│ potatoes             │  75820 │
│ carrots              │  75458 │
│ pineapple            │  74345 │
│ soy sauce            │  70355 │
│ black pepper         │  69064 │
│ thyme                │  68429 │
│ mustard              │  65948 │
│ chicken broth        │  65112 │
│ bacon                │  64956 │
│ honey                │  64626 │
│ oregano              │  64077 │
│ ground beef          │  64068 │
│ unsalted butter      │  63848 │
│ mushrooms            │  61465 │
│ Worcestershire sauce │  59328 │
│ cornstarch           │  58476 │
│ green pepper         │  58388 │
│ Cheddar cheese       │  58354 │
└──────────────────────┴────────┘

50 rows in set. Elapsed: 0.224 sec. Processed 2.23 million rows, 361.57 MB (9.97 million rows/s., 1.62 GB/s.)
Peak memory usage: 97.00 MiB.


-- самые сложные рецепты с сельдереем
nicshal-ubuntu :) SELECT
    title,
    length(NER),
    length(directions)
FROM recipes
WHERE has(NER, 'celery')
ORDER BY length(directions) DESC
LIMIT 10;

SELECT
    title,
    length(NER),
    length(directions)
FROM recipes
WHERE has(NER, 'celery')
ORDER BY length(directions) DESC
LIMIT 10

Query id: d8346c63-b5ff-4907-b5c7-89bbc6fc8b7f

┌─title────────────────────────────────────────────────────────────────────────┬─length(NER)─┬─length(directions)─┐
│ Shell's Potato Soup With Carrots                                             │          13 │                128 │
│ Roast Turkey and Pan Sauce                                                   │          34 │                113 │
│ Harlequin Soup                                                               │          15 │                 99 │
│ Perfect Roast Turkey                                                         │          39 │                 89 │
│ ChuckwagonCookie's Authentic NC Style Pork BBQ (Oven or Grill)               │          21 │                 87 │
│ Dry-Brined Roasted Turkey with Sage and Cider Gravy                          │          24 │                 84 │
│ Cabbage Leaf Of Beef Ragout With Crisped Root Vegetables Recipe              │          19 │                 78 │
│ Butter Poached Lobster Ravioli                                               │          31 │                 77 │
│ Veal Osso Buco with Saffron Risotto, English Peas, and Pea Shoots            │          29 │                 74 │
│ Braised Beef Brisket with Beluga Lentils, Horseradish Cream, and Salsa Verde │          33 │                 71 │
└──────────────────────────────────────────────────────────────────────────────┴─────────────┴────────────────────┘

10 rows in set. Elapsed: 0.658 sec. Processed 2.23 million rows, 1.64 GB (3.39 million rows/s., 2.49 GB/s.)
Peak memory usage: 109.10 MiB.


-- шаги по приготовлению макаронов с мидиями
nicshal-ubuntu :) SELECT arrayJoin(directions)
FROM recipes
WHERE title = 'pasta with mussels';

SELECT arrayJoin(directions)
FROM recipes
WHERE title = 'pasta with mussels'

Query id: 438ff7db-a4e0-489b-bd41-37dd9c0d1e41

┌─arrayJoin(directions)──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│ start with washing the mussels (cutting the beard,scraping the barnacles off using a knife or metal sponge ,with cold water and then rinse ) throw out the ones that are open or their shell is broken that means they are not alive any more ;)           │
│ take 2 of the garlic cloves and chop them into 8 pieces (you can crush them if you like crushed ) chop the parsley in little pieces ,put some olive oil in a pan,after the oil gets hot ,then put most of the parsley in the pan (keep some for garnish) after 1 minute (depending on your stove,as soon as the parsley starts to get fried) add the garlic and stir well, add the tomato juice as soon as the garlic starts to change color,add 1/4 cup of water and let the sauce boil add the lemon zest and a spoon of white wine ,you can add salt black pepper and the peperoncini (spicy) whenever & as much as you like. │
│ boil water for pasta, after it starts boiling add 1,5 teaspoon of sea salt and then add the pasta (follow the inscription on the box)                                                                                                                      │
│ either in a deep pan,or a pot ,put about 2 tablespoons of olive oil ,add the 2 other garlic cloves and chop each in half (don't crush) as soon as the garlic starts changing color add the mussels in the pot (add 2 tablespoons of white wine and a teaspoon of lemon juice) and put the lid. │
│ the mussels are ready when they all open, take them out of the pot (it's better if you Don't drain them the juice left in the pot can be added to your tomato sauce.)                                                                                      │
│ after you take out the mussels (You Can Also separate them from their shell after they're done) put them in the tomato sauce and start swirling them and mix them so the mussels get mixed with the sauce add some of the water (the water remaining after you cooked the mussels) to the mix and put the lid on top of the pan so they get together well . │
│ your pasta must have been ready by now, add the pasta into the pan with sauce and mussels ,mix so the sauce and the pasta also get together!                                                                                                               │
│ and you're ready to serve buon appetito !                                                                                                                                                                                                                  │
└────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘

8 rows in set. Elapsed: 0.007 sec. Processed 34.64 thousand rows, 6.82 MB (4.85 million rows/s., 956.06 MB/s.)
Peak memory usage: 11.90 MiB.

```