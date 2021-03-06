# Домашние задания по курсу MongoDB (_OTUS_)

## Список домашних заданий
1. [CAP теорема](#cap)
2. [Варианты установки MongoDB](#install)
3. [Базовые понятия MongoDB, CRUD, фильтры ](#base)

------
------

_<a name="cap"><h3>ДЗ 1: CAP теорема</h3></a>_

### Цель:
В результате выполнения ДЗ вы научитесь работать с гитом.
Необходимо написать к каким системам по CAP теореме относится MongoDB.
ДЗ сдается ссылкой на гит, где расположен миниотчет в маркдауне.

### Критерии оценки:
* задание выполнено - 10 баллов
* предложено красивое решение - плюс 2 балла
* предложено рабочее решение, но не устранены недостатки, указанные преподавателем - минус 2 балла

------

### CAP теорема

**CAP – это акроним** от англоязычных слов Consistency (Согласованность, Целостность), Availability (Доступность) и Partition tolerance (Устойчивость к разделению). Согласно утверждению профессора Калифорнийского университета в Беркли, Эрика Брюера, сделанному в 2000-м году, в распределенных системах осуществимы лишь 2 свойства из указанных 3-х. В частности, считается что нереляционные базы данных жертвуют согласованностью данных в пользу доступности и устойчивости к разделению, когда расщепление распределённой системы на несколько изолированных частей сохраняет корректный отклик от каждой из них.

<p align="center"><img src="hw1\cap_0.png" /></p>

<p>:heavy_check_mark: CA (Availability + Consistency – Parition tolerance), когда данные во всех узлах кластера согласованы и доступны, но не устойчивы к разделению. Это означает, что реплики одной и той же информации, распределенные по разным серверам друг другу, не противоречат друг другу и любой запрос к распределённой системе завершается корректным откликом. Такие системы возможны при поддержке ACID-требований к транзакциям (Атомарность, Согласованность, Изоляция, Долговечность) и абсолютной надежности сети. На практике таких решений на основе кластерных систем управления базами данных почти не существует. Классическим примером CA-системы называют распределённую службу каталогов LDAP, а также реляционные базы данных (PostgreSQL, MySQL, MariaDB, MS SQL Server).</p>
<p>:heavy_check_mark: CP-система (Consistency + Partition tolerance – Availability) в каждый момент обеспечивает целостность данных и способна работать в условиях распада в ущерб доступности, не выдавая отклик на запрос. Устойчивость к разделению требует дублирования изменений во всех узлах системы, что реализуется с помощью распределённых пессимистических блокировок для сохранения целостности. По сути, CP – это система с несколькими синхронно обновляемыми мастер-базами. Она всегда корректна, отрабатывая транзакцию, только в том случае, если изменения удалось распространить по всем серверам. Она продолжает корректно читать данные даже при отказе одного из узлов кластера. Но в этом случае запись будет обрываться или сильно задерживаться, пока система не убедится в своей целостности и согласованности (консистентности). Из NoSQL-СУБД к CP-системам принято относить Apache HBase, MongoDB, Redis, MemcasheDB, Berkley DB, HyperTable и Google Big Table. </p>
<p>:heavy_check_mark: AP-система (Availability + Partition tolerance – Consistency) не гарантирует целостность данных, обеспечивая их доступность и устойчивость к разделению, например, как в распределённых веб-кэшах и DNS. Считается, что большинство NoSQL-СУБД относятся к этому классу систем, обеспечивая лишь некоторой уровень согласованности данных в конечном счете (eventually consistent). Таким образом, AP-система может быть представлена кластером из нескольких узлов, каждый из которых может принимать данные, но не обязуется в тот же момент распространять их на другие сервера. Такая система отлично справляется с отказами нескольких узлов, но, когда они снова начинают работать, возможна выдача пользователям старых данных. К AP-системам относят CoucheDB, Cassandra, Riak, Amazon DynamoDB.</p>

При всей понятной на первый взгляд концепции тройственной ограниченности, CAP-теорему критикуют за **чрезмерное упрощение** важных понятий, что приводит к неверному пониманию первоначального смысла модели. В результате этого теорема из строгого, математически доказанного утверждения превращается в маркетинговый термин с расплывчатым смыслом. Наиболее явно это выражается в следующих случаях:

- **Согласованность (Consistency)** в САР означает линеаризуемость, а не фиксацию завершенной транзакции, как в ACID. Напомним, линеаризуемость – это локальное и неблокируемое свойство программы, при котором результат любого параллельного выполнения операций эквивалентен их последовательному выполнению. Для любого другого потока выполнение линеаризуемой операции является мгновенным: операция либо не начата, либо завершена. В реальности это значит, что информация во всех репликах, включая кэшированные данные, должна быть одна и та же. Достичь этого не очень просто.
- Определение **доступности (Availability)** ничего не говорит про временную задержку обработки данных (latency), при большом значении которой систему сложно назвать доступной на практике.
- **Устойчивость к разделению (Partition tolerance)** означает, что для связи используется асинхронная сеть, которая может терять или задерживать сообщения, что характерно для любой Интернет-системы. Поэтому создается ложное впечатление возможности настраивать параметр «P» в распределенных системах и рассматривать локальные хранилища данных в нерелевантном контексте.

В связи с этим, невозможно просто сказать, что **MongoDB** - это CP/AP/CA, потому что на самом деле это компромисс между C, A и P в зависимости конфигурации уровней согласованности:

| Схема реализации                       | Главная цель | Описание                                                       |
|----------------------------------------|--------------|----------------------------------------------------------------|
| Не распределенная                      | CA           | Система доступна и консистентна                                |
| Распределенная, majority read-write    | CP           | Запись и чтение только после подтверждения большинством реплик |
| Распределенная, no majority read-write | AP           | Записи из old primary могут быть отброшены                     |

-----

### Tеорема PACELC

Еще одним альтернативным подходом к построению распределенных систем считается **теорема PACELC**, впервые описанная Даниелом Дж. Абади из Йельского университета в 2012 году. 

Она основана на модели CAP, но, помимо согласованности, доступности и устойчивости к разделению также включает временную задержку (L, Latency) и логическое исключение между сочетаниями этих понятий. 

Согласно PACELC, в случае сетевого разделения (P) в распределенной системе необходимо выбирать между доступностью (A) и согласованностью (C), как и в CAP- теореме, но в остальном (E, ELSE), даже при нормальной работе системы без разделения, нужно выбирать между задержкой (L) и согласованностью (C). 

В логическом выражении PACELC формулируют следующим образом:  **IF P -> (C or A), ELSE (C or L)**. 

Так PACELC расширяет и уточняет CAP-теорему, регламентируя необходимость поиска компромисса между временной задержкой и согласованностью данных в распределенных Big Data системах.


<p align="center"><img src="hw1\cap_02.png" /></p>


:heavy_check_mark: **MongoDB** можно рассматривать как **PC/EC-базу**, которая в большинстве случаев гарантирует, согласованность операций чтения и записи.

Использованные ресурсы:
1. [MongoDB documentation. Causal consistency read write concerns](https://docs.mongodb.com/manual/core/causal-consistency-read-write-concerns/) 
2. [Bigdata school: CAP-теорема](https://www.bigdataschool.ru/wiki/cap)
3. [Bigdata school: Критика CAP-теоремы. Теорема PACELC](https://www.bigdataschool.ru/blog/cap-alternatives-for-nosql-and-big-data.html)

-----
-----

_<a name="install"><h3>ДЗ 2: Варианты установки MongoDB</h3></a>_
_Ход выполнения:_

### Создание и настройка среды в Google Cloud Platform

1. Был создан проект в Google Cloud Platform: [**mongodb2021-19930326**](https://console.cloud.google.com/compute/instances?authuser=1&orgonly=true&project=otus-edu&supportedpurview=organizationId)
2. Установлен на рабочей машине **gcloud**
3. Подключение инстанса VM (ОС Ubuntu 20.04) к созданному проекту через gcloud beta compute
4. Для подключения к виртуальной машине используем коменда ssh gcp :
   - ssh- ключ был автоматически подгружен в метадату GCP и на саму виртуальную машину при подключении VM инстанса через gcloud
   - добавлен алиас gcp

### Установка инстанса **Mongodb v4.4** :
1) Загрузка и установка стабильной версиии пакета MongoDB:
   ```console
   wget -qO - https://www.mongodb.org/static/pgp/server-4.4.asc | sudo apt-key add - && echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/4.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.4.list && sudo apt-get update && sudo apt-get install -y mongodb-org
   ```
2) Создание спейсов для хранения данных и конфигураций MongoDB:
   ```console
   sudo mkdir /home/dinora_ianbarisova/mongo && sudo mkdir /home/dinora_ianbarisova/mongo/db1 && sudo chmod 777 /home/dinora_ianbarisova/mongo/db1
   ```
3) Форк демона MongoDB:
   ```console
   mongod --dbpath /home/dinora_ianbarisova/mongo/db1 --port 27001 --fork --logpath /home/dinora_ianbarisova/mongo/db1/db1.log --pidfilepath /home.dinora_ianbarisova/mongo/db1/db1.pid
   ``` 
4) Подключение к БД:
   ```console
   mongo --port 27001
   ```
5) Создание рутового пользователя:
   ```console
    use admin;
    db.createUser( { user: "root", pwd: "otus$123", roles: [ "userAdminAnyDatabase", "dbAdminAnyDatabase", "readWriteAnyDatabase" ] } )
   ```
6) Останавливаем демона монги и запускаем заново, но с флагом требования аутентификации:
   ```console
   mongod --dbpath /home/mongo/db1 --port 27001 --fork --logpath /home/mongo/db1/db1.log --pidfilepath /home/mongo/db1/db1.pid --bind_ip_all --auth
   ```
7) Команда для подключения к текущему сервису монги:
   ```console
   mongo -u root -p otus\$123 --authenticationDatabase admin --port 27001
   ```
### Запуск **MongoDB v5.0** в docker-контейнере

1) Установка docker:
   ```console
   sudo apt-get update && sudo apt-get install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common && curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add - && sudo apt-key fingerprint 0EBFCD88 && sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" && sudo apt-get update && sudo apt-get install -y docker-ce docker-ce-cli containerd.io
   ```
2) Установка docker-compose:
   ```console
   sudo apt install docker-compose -u
   ```
3) Копирование файл docker-compose.yaml на сервер: scp .\docker-compose.csv gcp:~/docker/mongo
   ```yaml
   version: '3.3'

   services:
      mongodb:
          image: mongo:5.0.4
          restart: always
          environment:
              MONGO_INITDB_ROOT_USERNAME: root
              MONGO_INITDB_ROOT_PASSWORD: otus$$123
          volumes:
            - /home/dinora_ianbarisova/docker/mongo/db:/data/db
          ports:
            - 27017:27017
   ```
4) Запуск контейнера с MongoDB: sudo docker-compose up -d
5) Открываем порты (27001, 27017) на виртуалке в настройках firewall проекта в GCP
6) Теперь можно подключиться из сети напрямую к MongoDB, запущенном на севере и в docker-контейнере, по ip VM:
   ```console
   mongo 34.88.161.216:27001 -u root -p otus\$123 --authenticationDatabase admin
   mongo 34.88.161.216:27017 -u root -p otus\$123 --authenticationDatabase admin
   ```

-----
-----

_<a name="base"><h3>ДЗ 3: Базовые понятия MongoDB, CRUD, фильтры</h3></a>_
_Ход выполнения:_

1. Копирование датасета на сервер:
   ```console
   scp .\books.csv gcp:~/hw/hw3
   ````
2. Импорт данных при помощи команды mongoimport:
   ```console
   mongoimport --type=json --file=books.json -d test_db -c books -u root -p otus\$123 --authenticationDatabase admin mongodb://localhost:27017
   ```
3. Примеры выполненных запросов:
   READ:
   1) ```console
      db.books.find( { "publishedDate" : { $gte : ISODate("2013-12-12T23:59:59.999Z") }},  {"title" : 1}).count() 
      ```
      [Result](hw3/result1.json)
   2) ```console
      db.books.distinct('authors.2', {$or: [ {$or: [{pageCount: {$lt :200}}, {pageCount :{$gt: 600}}]}, {status: 'MEAP'}]})
      ```
      [Result](hw3/result2.json)
   3) ```console
      db.books.createIndex({"title": "text"})
      db.books.find({'$text': { '$search': "Java"}}, fields=( { '_id': 0, 'title': 1, 'score': { '$meta': 'textScore' }} )).sort({score: { '$meta' : "textScore"}})
      ```
      [Result](hw3/result3.json)

   UPDATE:
   1) ```console
      db.books.updateMany({"status": 'PUBLISH'}, {$set: {profit: Math.random()*100}, $setOnInsert: {application: 'yes'}}, {upsert: true})
      ```
      [Result](hw3/result4.json)
   2) ```console
      db.books.update({}, {$set: {"authors.$[element]": 'Ianbarisova Dinora'}}, {multi: true, arrayFilters: [{"element": {$eq: ""}}]})
      ```
      [Result](hw3/result5.json)

   DELETE:
   1) ```console
      db.books.deleteMany( {'categories' : 'PHP'})
      ```
      [Result](hw3/result6.json)
   2) ```console
      db.books.findOneAndDelete( {'categories' : 'Perl'})
      ```
      [Result](hw3/result7.json)