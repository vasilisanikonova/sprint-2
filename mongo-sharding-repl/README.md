# mongo-sharding-repl

## Как запустить

1. Запуск mongodb и приложения

```shell
docker-compose up -d
```


2. Инициализация БД

```shell
docker exec -it configServer mongosh

rs.initiate(
  {
    _id : "config_server",
       configsvr: true,
    members: [
      { _id : 0, host : "configServer" }
    ]
  }
);
exit();
```

3. Подключение к первой реплике

```shell
docker exec -it repl11 mongosh

rs.initiate({_id: "rs0", members: [{_id: 0, host: "repl11"}, {_id: 1, host: "repl12"}, {_id: 2, host: "repl13"}]});
exit();

```
```shell
docker exec -it repl21 mongosh

rs.initiate({_id: "rs1", members: [{_id: 0, host: "repl21"},{_id: 1, host: "repl22"},{_id: 2, host: "repl23"}]});
exit();
```
4. Инициализация роутера и наполнение данными:
```shell
docker exec -it mongos_router1 mongosh

sh.addShard( "rs0/repl11,repl12,repl13" );
sh.addShard( "rs1/repl21,repl22,repl23" );

sh.enableSharding("somedb");
sh.shardCollection("somedb.helloDoc", { "name" : "hashed" } )

use somedb
for(var i = 0; i < 1000; i++) db.helloDoc.insert({age:i, name:"testname"+i})
db.helloDoc.countDocuments() 
exit();
```
5. Проверка по первой реплике:
```shell
docker compose exec -T repl11 mongosh
use somedb
db.helloDoc.countDocuments();
exit();
 ```
```shell
docker compose exec -T repl12 mongosh
use somedb
db.helloDoc.countDocuments();
exit();
 ```
```shell
docker compose exec -T repl13 mongosh
use somedb
db.helloDoc.countDocuments();
exit();
 ```
```shell
docker compose exec -T repl21 mongosh
use somedb
db.helloDoc.countDocuments();
exit();
 ```
```shell
docker compose exec -T repl22 mongosh
use somedb
db.helloDoc.countDocuments();
exit();
 ```
```shell
docker compose exec -T repl23 mongosh
use somedb
db.helloDoc.countDocuments();
exit();
 ```