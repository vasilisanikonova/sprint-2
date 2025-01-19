# mongo-sharding

## Как запустить

1. Запуск mongodb и приложения

```shell
docker-compose up -d
```

2. Инициализация БД

```shell
docker exec -it configServer mongosh --port 27020

rs.initiate(
  {
    _id : "config_server",
       configsvr: true,
    members: [
      { _id : 0, host : "configServer:27020" }
    ]
  }
);
exit();
```
3. Инициализация шардов
```shell
docker exec -it shard-1 mongosh --port 27021

rs.initiate(
    {
      _id : "shard-1",
      members: [
        { _id : 0, host : "shard1:27021" }
      ]
    }
);
exit();
```
```shell
docker exec -it shard-2 mongosh --port 27022

rs.initiate(
    {
      _id : "shard-2",
      members: [
        { _id : 1, host : "shard2:27022" }
      ]
    }
  );

rs.status();
exit();
```
4. Инициализация роутера и наполнение данными:
```shell
docker exec -it mongos_router1 mongosh --port 27023

sh.addShard( "shard-1/shard1:27021");
sh.addShard( "shard-2/shard2:27022");

sh.enableSharding("somedb");
sh.shardCollection("somedb.helloDoc", { "name" : "hashed" } )

use somedb
for(var i = 0; i < 1000; i++) db.helloDoc.insert({age:i, name:"testname"+i})
db.helloDoc.countDocuments() 
exit();
```
5. Проверка по первой шарде
```shell
 docker exec -it shard-1 mongosh --port 27021
 use somedb
 db.helloDoc.countDocuments()
 exit();
 ```
5. Проверка по второй шарде
```shell
 docker exec -it shard-2 mongosh --port 27022
 use somedb;
 db.helloDoc.countDocuments();
 exit();
 ```