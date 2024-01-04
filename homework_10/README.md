## Описание/Пошаговая инструкция выполнения домашнего задания:
> -  Разворачиваем кластер Etcd любым способом. Проверяем отказоустойчивость - 10 баллов
> -  Разворачиваем кластер Consul любым способом. Проверяем отказоустойчивость - 10 баллов

## Описание процесса поднятия кластера etcd с помощью Docker Compose
Разворачиваться будет только кластер etcd
Эксперименты проводил в Idea. Создал новый проект Docker-test. В проекте сделал папку deploy.
В папку deploy положил docker-compose-etcd.yml:
```
version: '2'
networks:
  byfn:

services:
  etcd1:
    image: quay.io/coreos/etcd
    container_name: etcd1
    command: etcd -name etcd1 -advertise-client-urls http://0.0.0.0:2379 -listen-client-urls http://0.0.0.0:2379 -listen-peer-urls http://0.0.0.0:2380 -initial-cluster-token etcd-cluster -initial-cluster "etcd1=http://etcd1:2380,etcd2=http://etcd2:2380,etcd3=http://etcd3:2380" -initial-cluster-state new
    ports:
      - 2379
      - 2380
    networks:
      - byfn

  etcd2:
    image: quay.io/coreos/etcd
    container_name: etcd2
    command: etcd -name etcd2 -advertise-client-urls http://0.0.0.0:2379 -listen-client-urls http://0.0.0.0:2379 -listen-peer-urls http://0.0.0.0:2380 -initial-cluster-token etcd-cluster -initial-cluster "etcd1=http://etcd1:2380,etcd2=http://etcd2:2380,etcd3=http://etcd3:2380" -initial-cluster-state new
    ports:
      - 2379
      - 2380
    networks:
      - byfn

  etcd3:
    image: quay.io/coreos/etcd
    container_name: etcd3
    command: etcd -name etcd3 -advertise-client-urls http://0.0.0.0:2379 -listen-client-urls http://0.0.0.0:2379 -listen-peer-urls http://0.0.0.0:2380 -initial-cluster-token etcd-cluster -initial-cluster "etcd1=http://etcd1:2380,etcd2=http://etcd2:2380,etcd3=http://etcd3:2380" -initial-cluster-state new
    ports:
      - 2379
      - 2380
    networks:
      - byfn
```

Стартовал docker-compose-etcd.yml.
Проверка - зашел на терминал в одном из поднятых контейнеров. Выполнил:
```
/ # etcdctl member list
ade526d28b1f92f7: name=etcd1 peerURLs=http://etcd1:2380 clientURLs=http://0.0.0.0:2379 isLeader=true
bd388e7810915853: name=etcd3 peerURLs=http://etcd3:2380 clientURLs=http://0.0.0.0:2379 isLeader=false
d282ac2ce600c1ce: name=etcd2 peerURLs=http://etcd2:2380 clientURLs=http://0.0.0.0:2379 isLeader=false

/ # etcdctl cluster-health
member ade526d28b1f92f7 is healthy: got healthy result from http://0.0.0.0:2379
member bd388e7810915853 is healthy: got healthy result from http://0.0.0.0:2379
member d282ac2ce600c1ce is healthy: got healthy result from http://0.0.0.0:2379
```

Сохранил несколько значений по ключу
```
/ # etcdctl set /foo test
test
/ # etcdctl set /foo1 test
test
/ # etcdctl set /foo3 test3
test3
/ # etcdctl get /foo3 
test3
```

Сейчас лидером является нода etcd1. Останавливаем контейнер etcd1. Проверяем работоспособность кластера
```
/ # etcdctl member list
ade526d28b1f92f7: name=etcd1 peerURLs=http://etcd1:2380 clientURLs=http://0.0.0.0:2379 isLeader=false
bd388e7810915853: name=etcd3 peerURLs=http://etcd3:2380 clientURLs=http://0.0.0.0:2379 isLeader=false
d282ac2ce600c1ce: name=etcd2 peerURLs=http://etcd2:2380 clientURLs=http://0.0.0.0:2379 isLeader=true

/ # etcdctl get /foo3 
test3
```

Видим, что лидером стала нода etcd2. Кластер корректно отвечает на запросы данных
Вновь стартовал ранее остановленную ноду etcd1. Лидер не поменялся
```
/ # etcdctl member list
ade526d28b1f92f7: name=etcd1 peerURLs=http://etcd1:2380 clientURLs=http://0.0.0.0:2379 isLeader=false
bd388e7810915853: name=etcd3 peerURLs=http://etcd3:2380 clientURLs=http://0.0.0.0:2379 isLeader=false
d282ac2ce600c1ce: name=etcd2 peerURLs=http://etcd2:2380 clientURLs=http://0.0.0.0:2379 isLeader=true
```