## Описание/Пошаговая инструкция выполнения домашнего задания:
> - Развернуть Instance ES – желательно в AWS
> - Создать в ES индекс, в нём должно быть обязательное поле text типа string
> - Создать для индекса pattern
> - Добавить в индекс как минимум 3 документа желательно со следующим содержанием:
    «моя мама мыла посуду а кот жевал сосиски»,
    «рама была отмыта и вылизана котом»,
    «мама мыла раму»
> - Написать запрос нечеткого поиска к этой коллекции документов ко ключу «мама ела сосиски»


## Установка ElasticSearch и Kibana

```sh
-- так не прокатило
curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch |sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list

-- через yandex
nicshal@nicshal-ubuntu:/etc$ echo "deb [trusted=yes] https://mirror.yandex.ru/mirrors/elastic/8/ stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list
nicshal@nicshal-ubuntu:/etc$ sudo apt update
nicshal@nicshal-ubuntu:/etc$ sudo apt install elasticsearch

-- ограничиваем количество памяти, потребляемой Elasticsearch
nicshal@nicshal-ubuntu:/etc$ sudo nano /etc/elasticsearch/jvm.options
-Xms512m
-Xmx512m

-- меняем IP
nicshal@nicshal-ubuntu:/etc$ sudo nano /etc/elasticsearch/elasticsearch.yml
http.host: 127.0.0.1

-- сбрасываем и устанавливаем пароль для пользователя elastic 
nicshal@nicshal-ubuntu:/etc$ sudo /usr/share/elasticsearch/bin/elasticsearch-reset-password -i -u elastic

nicshal@nicshal-ubuntu:/etc$ sudo systemctl start elasticsearch

nicshal@nicshal-ubuntu:/etc$ sudo systemctl status elasticsearch
● elasticsearch.service - Elasticsearch
     Loaded: loaded (/lib/systemd/system/elasticsearch.service; disabled; vendor preset: enabled)
     Active: active (running) since Fri 2023-12-29 01:46:41 MSK; 39s ago
       Docs: https://www.elastic.co
   Main PID: 248991 (java)
      Tasks: 98 (limit: 18896)
     Memory: 939.8M
     CGroup: /system.slice/elasticsearch.service
             ├─248991 /usr/share/elasticsearch/jdk/bin/java -Xms4m -Xmx64m -XX:+UseSerialGC -Dcli.name=server -Dcli.script=/usr/share/elasticsearch/bin/elasticsearch -Dcli.libs=lib/tools/server-cli -Des>
             ├─249051 /usr/share/elasticsearch/jdk/bin/java -Des.networkaddress.cache.ttl=60 -Des.networkaddress.cache.negative.ttl=10 -Djava.security.manager=allow -XX:+AlwaysPreTouch -Xss1m -Djava.awt>
             └─249079 /usr/share/elasticsearch/modules/x-pack-ml/platform/linux-x86_64/bin/controller

дек 29 01:46:31 nicshal-ubuntu systemd[1]: Starting Elasticsearch...
дек 29 01:46:32 nicshal-ubuntu systemd-entrypoint[248991]: дек 29, 2023 1:46:32 AM sun.util.locale.provider.LocaleProviderAdapter <clinit>
дек 29 01:46:32 nicshal-ubuntu systemd-entrypoint[248991]: WARNING: COMPAT locale provider will be removed in a future release
дек 29 01:46:41 nicshal-ubuntu systemd[1]: Started Elasticsearch.

-- Для того чтобы убедится что Elasticsearch работает, выполняем такой запрос с помощью curl:
nicshal@nicshal-ubuntu:/etc$ curl -XGET https://127.0.0.1:9200 --insecure -u "elastic:xxxxxxxx"
{
  "name" : "nicshal-ubuntu",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "HdzWnmblSbqiCZJyufnyiQ",
  "version" : {
    "number" : "8.10.3",
    "build_flavor" : "default",
    "build_type" : "deb",
    "build_hash" : "c63272efed16b5a1c25f3ce500715b7fddf9a9fb",
    "build_date" : "2023-10-05T10:15:55.152563867Z",
    "build_snapshot" : false,
    "lucene_version" : "9.7.0",
    "minimum_wire_compatibility_version" : "7.17.0",
    "minimum_index_compatibility_version" : "7.0.0"
  },
  "tagline" : "You Know, for Search"
}

-- ставим Kibana
nicshal@nicshal-ubuntu:/etc$ sudo apt install kibana
nicshal@nicshal-ubuntu:/etc$ sudo systemctl status kibana
● kibana.service - Kibana
     Loaded: loaded (/lib/systemd/system/kibana.service; disabled; vendor preset: enabled)
     Active: active (running) since Fri 2023-12-29 01:57:28 MSK; 8s ago
       Docs: https://www.elastic.co
   Main PID: 249758 (node)
      Tasks: 11 (limit: 18896)
     Memory: 377.6M
     CGroup: /system.slice/kibana.service
             └─249758 /usr/share/kibana/bin/../node/bin/node /usr/share/kibana/bin/../src/cli/dist

дек 29 01:57:28 nicshal-ubuntu systemd[1]: Started Kibana.

-- Для Kibana надо сгенерировать токен доступа. Для этого выполните команду:
nicshal@nicshal-ubuntu:/usr/share/elasticsearch/bin$ sudo /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana
eyJ2ZXIiOiI4LjEwLjMiLCJhZHIiOlsiMTkyLjE2OC4xLjEwNzo5MjAwIl0sImZnciI6Ijg0OGFkZTE3NzY5NTk5MDI1ODg2M2MxNzIwNGJhNGQyYzFjMzkyMDYxZjJkZjI3MDdhZTM2MDhmZjExNWEzOGMiLCJrZXkiOiJqZG1wc293QlYzLU40djVFYm4yVDo3UXRvQWcxYlFUTy1KUi1BQWpMOVNBIn0=


-- заходим в браузер - localhost:5601
-- вводим сгенерировать токен доступа

-- Kibana запросит код верифркации. Генерируем
sudo /usr/share/kibana/bin/kibana-verification-code

-- далее Kibana делает настройки
-- далее вводим логин/пароль elastic/xxxxxx

```

## Создаем индекс, добавляем данные, осуществляем поиск
```sh
-- делаем индекс
PUT /otus_test
{
  "mappings": {
    "properties": {
      "title": { "type": "text" },
      "content": { "type": "text" }
    }
  }
}

{
  "acknowledged": true,
  "shards_acknowledged": true,
  "index": "otus_test"
}

-- создаем для индекса pattern (не стал показывать, просто)

-- добавляем данные в индекс
POST /otus_test/_doc/
{
  "title": "Первая запись",
  "content": "моя мама мыла посуду а кот жевал сосиски"
}

{
  "_index": "otus_test",
  "_id": "49mStIwBV3-N4v5EJX2d",
  "_version": 1,
  "result": "created",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 0,
  "_primary_term": 1
}

POST /otus_test/_doc/
{
  "title": "Вторая запись",
  "content": "рама была отмыта и вылизана котом"
}

{
  "_index": "otus_test",
  "_id": "5NmTtIwBV3-N4v5EvX0y",
  "_version": 1,
  "result": "created",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 1,
  "_primary_term": 1
}

POST /otus_test/_doc/
{
  "title": "Третья запись",
  "content": "мама мыла раму"
}

{
  "_index": "otus_test",
  "_id": "5dmUtIwBV3-N4v5EFX2-",
  "_version": 1,
  "result": "created",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 2,
  "_primary_term": 1
}

-- делаем поисковый запрос
GET /otus_test/_search
{
  "query": {
    "match": {
      "content": 
        "мама ела сосиски"
    }
  }
}

{
  "took": 1,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 2,
      "relation": "eq"
    },
    "max_score": 1.241674,
    "hits": [
      {
        "_index": "otus_test",
        "_id": "49mStIwBV3-N4v5EJX2d",
        "_score": 1.241674,
        "_source": {
          "title": "Первая запись",
          "content": "моя мама мыла посуду а кот жевал сосиски"
        }
      },
      {
        "_index": "otus_test",
        "_id": "5dmUtIwBV3-N4v5EFX2-",
        "_score": 0.5820575,
        "_source": {
          "title": "Третья запись",
          "content": "мама мыла раму"
        }
      }
    ]
  }

```