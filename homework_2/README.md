# Описание/Пошаговая инструкция выполнения домашнего задания:
> - установить MongoDB одним из способов: ВМ, докер;
> - заполнить данными;
> - написать несколько запросов на выборку и обновление данных
> - создать индексы и сравнить производительность

# Описание процесса установки (локальная установка)
```sh
-- чистим предыдушее
sudo apt-get purge mongodb-org*
sudo rm -r /var/log/mongodb
sudo rm -r /var/lib/mongodb

-- ставим новое
sudo apt-get update
sudo apt-get install -y mongodb-org
sudo service mongod start

-- смотрим статус 
sudo service mongod status

 mongod.service - MongoDB Database Server
     Loaded: loaded (/lib/systemd/system/mongod.service; enabled; vendor preset: enabled)
     Active: active (running) since Sat 2023-10-28 18:16:01 MSK; 24min ago
       Docs: https://docs.mongodb.org/manual
   Main PID: 28546 (mongod)
     Memory: 149.2M
     CGroup: /system.slice/mongod.service
             └─28546 /usr/bin/mongod --config /etc/mongod.conf

окт 28 18:16:01 nicshal-ubuntu systemd[1]: Started MongoDB Database Server.
окт 28 18:16:01 nicshal-ubuntu mongod[28546]: {"t":{"$date":"2023-10-28T15:16:01.387Z"},"s":"I",  "c":"CONTROL",  "id":7484500, "ctx":"main","msg":"Environ>

-- запускаем mongosh, смотрим версию
nicshal@nicshal-ubuntu:~$ mongosh
Current Mongosh Log ID:	653d2ca07fd73b2adc240a94
Connecting to:		mongodb://127.0.0.1:27017/?directConnection=true&serverSelectionTimeoutMS=2000&appName=mongosh+2.0.2
Using MongoDB:		7.0.2
Using Mongosh:		2.0.2

-- устанавливаем админа
use admin
db.createUser(
{
user: "admin",
pwd: "admin",
roles: [ { role: "userAdminAnyDatabase", db: "admin" }, "readWriteAnyDatabase" ]
}
)

-- редактируем конфигурацию
sudo vi /etc/mongod.conf
security:
    authorization: enabled

-- рестарт
sudo systemctl restart mongod

-- могут оказаться полезными сследующие команды
sudo systemctl unmask mongod
sudo service mongod start

```

Далее закачал тестовые dataset'ы с https://github.com/neelabalan/mongodb-sample-dataset и залил в базу
```
./script.sh localhost 27017 admin .......
```


# Примеры работы с базой
## Поиск данных
```
nicshal@nicshal-ubuntu:~$ mongosh --authenticationDatabase "sample_training" -u "work_user" -p 
Enter password: ******
Current Mongosh Log ID:	653d3ac3c5f999eec5abc8d2
Connecting to:		mongodb://<credentials>@127.0.0.1:27017/?directConnection=true&serverSelectionTimeoutMS=2000&authSource=sample_training&appName=mongosh+2.0.2
Using MongoDB:		7.0.2
Using Mongosh:		2.0.2

For mongosh info see: https://docs.mongodb.com/mongodb-shell/

test> use sample_training
switched to db sample_training
sample_training> db.tweets.find({id : 22821891000})
[
  {
    _id: ObjectId("5c8eccb1caa187d17ca66ea8"),
    text: 'Soms kun je ook te lang door gaan, nu wel klaar met werken.Nieuwe frontpage http://www.lustforlifemagazine.nl online sneller, harder, beter!',
    in_reply_to_status_id: null,
    retweet_count: null,
    contributors: null,
    created_at: 'Thu Sep 02 18:52:51 +0000 2010',
    geo: null,
    source: '<a href="http://www.twittergadget.com" rel="nofollow">tGadget</a>',
    coordinates: null,
    in_reply_to_screen_name: null,
    truncated: false,
    entities: {
      user_mentions: [],
      urls: [
        {
          indices: [ 76, 109 ],
          url: 'http://www.lustforlifemagazine.nl',
          expanded_url: null
        }
      ],
      hashtags: []
    },
    retweeted: false,
    place: null,
    user: {
      friends_count: 99,
      profile_sidebar_fill_color: 'efefef',
      location: 'Arnhem',
      verified: false,
      follow_request_sent: null,
      favourites_count: 0,
      profile_sidebar_border_color: 'eeeeee',
      profile_image_url: 'http://a2.twimg.com/profile_images/944911278/672476992_5_T3eH_normal.jpeg',
      geo_enabled: false,
      created_at: 'Thu May 15 17:04:44 +0000 2008',
      description: 'Programmeur en webmaster van www.lustforlifemagazine.nl en www.debassist.nl. Student, fanatiek sporter en wederhelft van @EvaHoefsloot.',
      time_zone: 'Amsterdam',
      url: 'http://www.jopperhebergen.nl',
      screen_name: 'jopperhebergen',
      notifications: null,
      profile_background_color: '131516',
      listed_count: 1,
      lang: 'en',
      profile_background_image_url: 'http://s.twimg.com/a/1282351897/images/themes/theme14/bg.gif',
      statuses_count: 1830,
      following: null,
      profile_text_color: '333333',
      protected: false,
      show_all_inline_media: false,
      profile_background_tile: true,
      name: 'Joppe Rhebergen',
      contributors_enabled: false,
      profile_link_color: '009999',
      followers_count: 122,
      id: 14788646,
      profile_use_background_image: true,
      utc_offset: 3600
    },
    favorited: false,
    in_reply_to_user_id: null,
    id: Long("22821891000")
  }
]
```

## Изменение данных
```
sample_training> db.tweets.updateOne({id : 22821891000}, {"$set" : {"user.friends_count" : 777}})
{
  acknowledged: true,
  insertedId: null,
  matchedCount: 1,
  modifiedCount: 1,
  upsertedCount: 0
}
sample_training> 
(To exit, press Ctrl+C again or Ctrl+D or type .exit)
sample_training> db.tweets.find({id : 22821891000})
[
  {
    _id: ObjectId("5c8eccb1caa187d17ca66ea8"),
    text: 'Soms kun je ook te lang door gaan, nu wel klaar met werken.Nieuwe frontpage http://www.lustforlifemagazine.nl online sneller, harder, beter!',
    in_reply_to_status_id: null,
    retweet_count: null,
    contributors: null,
    created_at: 'Thu Sep 02 18:52:51 +0000 2010',
    geo: null,
    source: '<a href="http://www.twittergadget.com" rel="nofollow">tGadget</a>',
    coordinates: null,
    in_reply_to_screen_name: null,
    truncated: false,
    entities: {
      user_mentions: [],
      urls: [
        {
          indices: [ 76, 109 ],
          url: 'http://www.lustforlifemagazine.nl',
          expanded_url: null
        }
      ],
      hashtags: []
    },
    retweeted: false,
    place: null,
    user: {
      friends_count: 777,
      profile_sidebar_fill_color: 'efefef',
      location: 'Arnhem',
      verified: false,
      follow_request_sent: null,
      favourites_count: 0,
      profile_sidebar_border_color: 'eeeeee',
      profile_image_url: 'http://a2.twimg.com/profile_images/944911278/672476992_5_T3eH_normal.jpeg',
      geo_enabled: false,
      created_at: 'Thu May 15 17:04:44 +0000 2008',
      description: 'Programmeur en webmaster van www.lustforlifemagazine.nl en www.debassist.nl. Student, fanatiek sporter en wederhelft van @EvaHoefsloot.',
      time_zone: 'Amsterdam',
      url: 'http://www.jopperhebergen.nl',
      screen_name: 'jopperhebergen',
      notifications: null,
      profile_background_color: '131516',
      listed_count: 1,
      lang: 'en',
      profile_background_image_url: 'http://s.twimg.com/a/1282351897/images/themes/theme14/bg.gif',
      statuses_count: 1830,
      following: null,
      profile_text_color: '333333',
      protected: false,
      show_all_inline_media: false,
      profile_background_tile: true,
      name: 'Joppe Rhebergen',
      contributors_enabled: false,
      profile_link_color: '009999',
      followers_count: 122,
      id: 14788646,
      profile_use_background_image: true,
      utc_offset: 3600
    },
    favorited: false,
    in_reply_to_user_id: null,
    id: Long("22821891000")
  }
]
```

## Создание индекса, сравнение времени поиска записи
> - индекса нет - значение totalDocsExamined: 24832. То есть обходит все документы
```
sample_training> db.tweets.find({id : 22821891000}).explain("executionStats")
{
  explainVersion: '2',
  queryPlanner: {
    namespace: 'sample_training.tweets',
    indexFilterSet: false,
    parsedQuery: { id: { '$eq': 22821891000 } },
    queryHash: '9B1B7FA3',
    planCacheKey: 'DE862384',
    maxIndexedOrSolutionsReached: false,
    maxIndexedAndSolutionsReached: false,
    maxScansToExplodeReached: false,
    winningPlan: {
      queryPlan: {
        stage: 'COLLSCAN',
        planNodeId: 1,
        filter: { id: { '$eq': 22821891000 } },
        direction: 'forward'
      },
      slotBasedPlan: {
        slots: '$$RESULT=s5 env: { s2 = Nothing (SEARCH_META), s1 = TimeZoneDatabase(Canada/Eastern...Europe/Lisbon) (timeZoneDB), s7 = 2.28219e+10, s3 = 1698512304416 (NOW) }',
        stages: '[1] filter {traverseF(s4, lambda(l1.0) { ((l1.0 == s7) ?: false) }, false)} \n' +
          '[1] scan s5 s6 none none none none lowPriority [s4 = id] @"5abfeff0-5f28-49f0-998f-632f80ef5d7c" true false '
      }
    },
    rejectedPlans: []
  },
  executionStats: {
    executionSuccess: true,
    nReturned: 1,
    executionTimeMillis: 30,
    totalKeysExamined: 0,
    totalDocsExamined: 24832,
    executionStages: {
      stage: 'filter',
      planNodeId: 1,
      nReturned: 1,
      executionTimeMillisEstimate: 17,
      opens: 1,
      closes: 1,
      saveState: 25,
      restoreState: 25,
      isEOF: 1,
      numTested: 24832,
      filter: 'traverseF(s4, lambda(l1.0) { ((l1.0 == s7) ?: false) }, false) ',
      inputStage: {
        stage: 'scan',
        planNodeId: 1,
        nReturned: 24832,
        executionTimeMillisEstimate: 17,
        opens: 1,
        closes: 1,
        saveState: 25,
        restoreState: 25,
        isEOF: 1,
        numReads: 24832,
        recordSlot: 5,
        recordIdSlot: 6,
        fields: [ 'id' ],
        outputSlots: [ Long("4") ]
      }
    }
  },
  command: {
    find: 'tweets',
    filter: { id: 22821891000 },
    '$db': 'sample_training'
  },
  serverInfo: {
    host: 'nicshal-ubuntu',
    port: 27017,
    version: '7.0.2',
    gitVersion: '02b3c655e1302209ef046da6ba3ef6749dd0b62a'[README.md](README.md)
  },
  serverParameters: {
    internalQueryFacetBufferSizeBytes: 104857600,
    internalQueryFacetMaxOutputDocSizeBytes: 104857600,
    internalLookupStageIntermediateDocumentMaxSizeBytes: 104857600,
    internalDocumentSourceGroupMaxMemoryBytes: 104857600,
    internalQueryMaxBlockingSortMemoryUsageBytes: 104857600,
    internalQueryProhibitBlockingMergeOnMongoS: 0,
    internalQueryMaxAddToSetBytes: 104857600,
    internalDocumentSourceSetWindowFieldsMaxMemoryBytes: 104857600,
    internalQueryFrameworkControl: 'trySbeEngine'
  },
  ok: 1
}

```

> - делаем индекс
```
sample_training> db.tweets.createIndex({"id" : 1})
id_1
```

> - еще раз делаем поиск. Видно, что обрабатывается один документ totalDocsExamined: 1. Значит индекс сработал
> - также видно, что значение executionTimeMillis: 3 стало меньше на порядок. То есть, время выполнение запроса уменьшилось на порядок после создания индекса
```
sample_training> db.tweets.find({id : 22821891000}).explain("executionStats")
{
  explainVersion: '2',
  queryPlanner: {
    namespace: 'sample_training.tweets',
    indexFilterSet: false,
    parsedQuery: { id: { '$eq': 22821891000 } },
    queryHash: '9B1B7FA3',
    planCacheKey: '6CADE188',
    maxIndexedOrSolutionsReached: false,
    maxIndexedAndSolutionsReached: false,
    maxScansToExplodeReached: false,
    winningPlan: {
      queryPlan: {
        stage: 'FETCH',
        planNodeId: 2,
        inputStage: {
          stage: 'IXSCAN',
          planNodeId: 1,
          keyPattern: { id: 1 },
          indexName: 'id_1',
          isMultiKey: false,
          multiKeyPaths: { id: [] },
          isUnique: false,
          isSparse: false,
          isPartial: false,
          indexVersion: 2,
          direction: 'forward',
          indexBounds: { id: [ '[22821891000.0, 22821891000.0]' ] }
        }
      },
      slotBasedPlan: {
        slots: '$$RESULT=s11 env: { s1 = TimeZoneDatabase(Canada/Eastern...Europe/Lisbon) (timeZoneDB), s6 = KS(2F0AA094D770FE04), s10 = {"id" : 1}, s2 = Nothing (SEARCH_META), s5 = KS(2F0AA094D7700104), s3 = 1698512677714 (NOW) }',
        stages: '[2] nlj inner [] [s4, s7, s8, s9, s10] \n' +
          '    left \n' +
          '        [1] cfilter {(exists(s5) && exists(s6))} \n' +
          '        [1] ixseek s5 s6 s9 s4 s7 s8 [] @"5abfeff0-5f28-49f0-998f-632f80ef5d7c" @"id_1" true \n' +
          '    right \n' +
          '        [2] limit 1 \n' +
          '        [2] seek s4 s11 s12 s7 s8 s9 s10 [] @"5abfeff0-5f28-49f0-998f-632f80ef5d7c" true false \n'
      }
    },
    rejectedPlans: []
  },
  executionStats: {
    executionSuccess: true,
    nReturned: 1,
    executionTimeMillis: 3,
    totalKeysExamined: 1,
    totalDocsExamined: 1,
    executionStages: {
      stage: 'nlj',
      planNodeId: 2,
      nReturned: 1,
      executionTimeMillisEstimate: 0,
      opens: 1,
      closes: 1,
      saveState: 0,
      restoreState: 0,
      isEOF: 1,
      totalDocsExamined: 1,
      totalKeysExamined: 1,
      collectionScans: 0,
      collectionSeeks: 1,
      indexScans: 0,
      indexSeeks: 1,
      indexesUsed: [ 'id_1' ],
      innerOpens: 1,
      innerCloses: 1,
      outerProjects: [],
      outerCorrelated: [ Long("4"), Long("7"), Long("8"), Long("9"), Long("10") ],
      outerStage: {
        stage: 'cfilter',
        planNodeId: 1,
        nReturned: 1,
        executionTimeMillisEstimate: 0,
        opens: 1,
        closes: 1,
        saveState: 0,
        restoreState: 0,
        isEOF: 1,
        numTested: 1,
        filter: '(exists(s5) && exists(s6)) ',
        inputStage: {
          stage: 'ixseek',
          planNodeId: 1,
          nReturned: 1,
          executionTimeMillisEstimate: 0,
          opens: 1,
          closes: 1,
          saveState: 0,
          restoreState: 0,
          isEOF: 1,
          indexName: 'id_1',
          keysExamined: 1,
          seeks: 1,
          numReads: 2,
          indexKeySlot: 9,
          recordIdSlot: 4,
          snapshotIdSlot: 7,
          indexIdentSlot: 8,
          outputSlots: [],
          indexKeysToInclude: '00000000000000000000000000000000',
          seekKeyLow: 's5 ',
          seekKeyHigh: 's6 '
        }
      },
      innerStage: {
        stage: 'limit',
        planNodeId: 2,
        nReturned: 1,
        executionTimeMillisEstimate: 0,
        opens: 1,
        closes: 1,
        saveState: 0,
        restoreState: 0,
        isEOF: 1,
        limit: 1,
        inputStage: {
          stage: 'seek',
          planNodeId: 2,
          nReturned: 1,
          executionTimeMillisEstimate: 0,
          opens: 1,
          closes: 1,
          saveState: 0,
          restoreState: 0,
          isEOF: 0,
          numReads: 1,
          recordSlot: 11,
          recordIdSlot: 12,
          seekKeySlot: 4,
          snapshotIdSlot: 7,
          indexIdentSlot: 8,
          indexKeySlot: 9,
          indexKeyPatternSlot: 10,
          fields: [],
          outputSlots: []
        }
      }
    }
  },
  command: {
    find: 'tweets',
    filter: { id: 22821891000 },
    '$db': 'sample_training'
  },
  serverInfo: {
    host: 'nicshal-ubuntu',
    port: 27017,
    version: '7.0.2',
    gitVersion: '02b3c655e1302209ef046da6ba3ef6749dd0b62a'
  },
  serverParameters: {
    internalQueryFacetBufferSizeBytes: 104857600,
    internalQueryFacetMaxOutputDocSizeBytes: 104857600,
    internalLookupStageIntermediateDocumentMaxSizeBytes: 104857600,
    internalDocumentSourceGroupMaxMemoryBytes: 104857600,
    internalQueryMaxBlockingSortMemoryUsageBytes: 104857600,
    internalQueryProhibitBlockingMergeOnMongoS: 0,
    internalQueryMaxAddToSetBytes: 104857600,
    internalDocumentSourceSetWindowFieldsMaxMemoryBytes: 104857600,
    internalQueryFrameworkControl: 'trySbeEngine'
  },
  ok: 1
}

```