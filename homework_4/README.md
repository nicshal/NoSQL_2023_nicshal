# Описание/Пошаговая инструкция выполнения домашнего задания:
> Развернуть кластер Couchbase
> Создать БД, наполнить небольшими тестовыми данными
> Проверить отказоустойчивость

# Описание процесса поднятия Couchbase Cluster с помощью Docker Compose
Эксперименты проводил в Idea. Создал новый проект Docker-test. В проекте сделал папку deploy.
В папку deploy положил docker-compose-couchbase.yml:
```
couchbase1:
  image: couchbase/server
  volumes:
    - /home/nicshal/IdeaProjects/Docker-test/couchbase/node1:/opt/couchbase/var
couchbase2:
  image: couchbase/server
  volumes:
    - /home/nicshal/IdeaProjects/Docker-test/couchbase/node2:/opt/couchbase/var
couchbase3:
  image: couchbase/server
  volumes:
    - /home/nicshal/IdeaProjects/Docker-test/couchbase/node3:/opt/couchbase/var
  ports:
    - 8091:8091
    - 8092:8092
    - 8093:8093
    - 11210:11210
```

Стартовал docker-compose-couchbase.yml.
Проверка:
```
nicshal@nicshal-ubuntu:~$ docker ps
CONTAINER ID   IMAGE              COMMAND                  CREATED          STATUS          PORTS                                                                                                                                                                          NAMES
cd5eed1f8d42   couchbase/server   "/entrypoint.sh couc…"   46 minutes ago   Up 46 minutes   8094-8097/tcp, 9123/tcp, 0.0.0.0:8091-8093->8091-8093/tcp, :::8091-8093->8091-8093/tcp, 11207/tcp, 11280/tcp, 0.0.0.0:11210->11210/tcp, :::11210->11210/tcp, 18091-18097/tcp   deploy_couchbase3_1
69c242cc4fdc   couchbase/server   "/entrypoint.sh couc…"   46 minutes ago   Up 46 minutes   8091-8097/tcp, 9123/tcp, 11207/tcp, 11210/tcp, 11280/tcp, 18091-18097/tcp                                                                                                      deploy_couchbase1_1
774a56804fa2   couchbase/server   "/entrypoint.sh couc…"   46 minutes ago   Up 46 minutes   8091-8097/tcp, 9123/tcp, 11207/tcp, 11210/tcp, 11280/tcp, 18091-18097/tcp                                                                                                      deploy_couchbase2_1

```

Определяем ip-адрес:
```
nicshal@nicshal-ubuntu:~$ docker-machine ip default
192.168.1.104
```

Идем в браузер - http://192.168.1.104:8091
В открывшейся странице нажимаем - Setup New Cluster
Вводим Cluster Name - nicshal-couchbase
Задаем логин - пароль

На следующей странице со всем соглашаемся. Жмем Configure Disk, Memory, Services

Определяем hostneme
```
nicshal@nicshal-ubuntu:~$ docker inspect --format '{{ .NetworkSettings.IPAddress }}' deploy_couchbase3_1
172.17.0.2
```
Добавляем этот адрес на страницу. Выбираем сервисы Data, Index, Query
Нажимаем Save&Finish

Далее определяем ip-адреса оставшихся двух серверов:
```
nicshal@nicshal-ubuntu:~$ docker inspect --format '{{ .NetworkSettings.IPAddress }}' deploy_couchbase2_1
172.17.0.4
nicshal@nicshal-ubuntu:~$ docker inspect --format '{{ .NetworkSettings.IPAddress }}' deploy_couchbase1_1
172.17.0.3
nicshal@nicshal-ubuntu:~$
```

Добавляем эти сервера через ADD SERVER. В одном из них (172.17.0.4) в дополнение к Data, Index, Query 
добавляем сервис Backup

Делаем ребалансировку - Rebalance

На вкладке Servers видим три сервера:
```
172.17.0.2 Group 1 data query index        0.7% 77.4% 44.2% --- 0/0 Statistics
172.17.0.3 Group 1 data query index        0.8% 77.5% 44.2% --- 0/0 Statistics
172.17.0.4 Group 1 data query index backup 0.6% 77.5% 44.2% --- 0/0 Statistics
```

Идем на вкладку Buckets. Нажимаем ссылку sample bucket. Выбираем travel-sample
Нажимаем Load Sample Data

Убеждаемся, что на вкладке Buckets появляется база travel-sample
```
travel-sample 63,288 100% 0 165MiB / 600MiB 133MiB Documents Scopes & Collections
```

Переходим на вкладку Query
Выполняем запрос:
```
SELECT route.airlineid, airline.name, route.sourceairport, route.destinationairport
FROM `travel-sample` route
INNER JOIN `travel-sample` airline
ON route.airlineid = META(airline).id
WHERE route.type = "route"
AND route.destinationairport = "SFO"
AND route.sourceairport = "BFL"
ORDER BY route.sourceairport;

-- результат
[
  {
    "airlineid": "airline_5209",
    "destinationairport": "SFO",
    "name": "United Airlines",
    "sourceairport": "BFL"
  }
]
```

Переходим на вкладку Servers. Удаляем сервер 172.17.0.3. Ждем, пока исчезнет 
Делаем ребалансировку

Повторяем запрос
```
SELECT route.airlineid, airline.name, route.sourceairport, route.destinationairport
FROM `travel-sample` route
INNER JOIN `travel-sample` airline
ON route.airlineid = META(airline).id
WHERE route.type = "route"
AND route.destinationairport = "SFO"
AND route.sourceairport = "BFL"
ORDER BY route.sourceairport;

-- результат
[
  {
    "airlineid": "airline_5209",
    "destinationairport": "SFO",
    "name": "United Airlines",
    "sourceairport": "BFL"
  }
]
```

Убедились, что кластер в рабочем состоянии

