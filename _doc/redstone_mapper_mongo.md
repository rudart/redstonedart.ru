---
layout: doc
menu_item: doc
title: redstone_mapper_mongo
prev: redstone_mapper
next: redstone_mapper_pg
---
[redstone_mapper_mongo](http://pub.dartlang.org/packages/redstone_mapper_mongo) расширяет [redstone_mapper](http://pub.dartlang.org/packages/redstone_mapper) и позволяет работать с MongoDB.

Этот пакет работает вокруг [mongo_dart](https://github.com/vadimtsushko/mongo_dart).

### Использование:

Создайте `MongoDbManager` для управления коллекциями базы данных:

```dart
var dbManager = new MongoDbManager("mongodb://localhost/dbname", poolSize: 3);
```

Если вы используете redstone_mapper как плагин Redstone.dart, то можете передать `MongoDbManager` в `getMapperPlugin()`, что сделает соединение к базе данных доступным для каждого запроса:

```dart
import 'package:redstone/server.dart' as app;
import 'package:redstone_mapper/plugin.dart';
import 'package:redstone_mapper_mongo/manager.dart';

main() {
  
  var dbManager = new MongoDbManager("mongodb://localhost/dbname", poolSize: 3);
  
  app.addPlugin(getMapperPlugin(dbManager));
  app.setupConsoleLog();
  app.start();
  
}

// redstone_mapper создаст атрибут "dbConn"
// для каждого запроса
@app.Route("/services/users/list")
listUsers(@app.Attr() MongoDb dbConn) =>
   dbConn.collection("users").find().toList();
   
// Если хотите, то можете создать геттер для доступа
// к соединению с базой для текущего запроса. Так вам не надо
// будет добавлять дополнительный параметр в каждый роут
MongoDb get mongoDb => app.request.attributes.dbConn;

```

Объект `MongoDb` предоставляет вспомогательные функции для кодирования и декодирования объектов:

```dart
import 'package:redstone/server.dart' as app;
import 'package:redstone_mapper/mapper.dart';
import 'package:redstone_mapper/plugin.dart';
import 'package:redstone_mapper_mongo/manager.dart';
import 'package:redstone_mapper_mongo/metadata.dart';

class User {
  
  // @Id - это специальная аннотация, которая берет поле "_id" документа
  // и говорит redstone_mapper, чтобы тот сконвертировал значение ObjectId в
  // строку и наоборот.
  @Id()
  String id;

  @Field()
  String username;

  @Field()
  String password;
  
}

MongoDb get mongoDb => app.request.attributes.dbConn;

@app.Route("/services/users/list")
@Encode()
Future<List<User>> listUsers() => 
  // Получим докуметны из коллекции "users", декодируем результат
  // в List<User>
  mongoDb.find("users", User); 

@app.Route("/services/users/add", methods: const[app.POST])
Future addUser(@Decode() User user) => 
  // запишем пользователя в коллекцию "users"
  mongoDb.insert("users", user);

```

Класс `MongoDb` не скрывает API пакета `mongo_dart`. Вы можете получить `DbCollection` через метод `MongoDb.collections()`. Вы можете получить оригинальный объект соединения через свойство `MongoDb.innderConn`.

Вы можете использовать `MongoDbService` для проведения операций над документами одного типа:

```dart

MongoDbService<User> userService = new MongoDbService<User>("users");

@app.Route("/services/users/list")
@Encode()
Future<List<User>> listUsers() => userService.find(); 

@app.Route("/services/users/add", methods: const[app.POST])
Future addUser(@Decode() User user) => userService.insert(user);

```

Также можно расширить `MongoDbService`:

```dart
@app.Group("/services/users")
Class UserService extends MongoDbService<User> {

  UserService() : super("users");

  @app.Route("/list")
  @Encode()
  Future<List<User>> list() => find();

  @app.Route("/add")
  Future add(@Decode() User user) => insert(user);

}
```

`MongoDbService` по-умолчанию использует соединение, связанное с текущим запросом. Если вы не используете Redstone.dart, убедитесь, что создаете новый сервис через `MongoDbService.fromConnection()`.