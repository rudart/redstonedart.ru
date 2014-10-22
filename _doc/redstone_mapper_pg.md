---
layout: doc
menu_item: doc
title: redstone_mapper_pg
prev: redstone_mapper_mongo
next: redstone_web_socket
---
[redstone_mapper_pg](http://pub.dartlang.org/packages/redstone_mapper_pg) расширяет [redstone_mapper](http://pub.dartlang.org/packages/redstone_mapper) и позволяет работать с PostgreSQL.

Этот пакет работает с драйвером [postgresql](https://github.com/xxgreg/postgresql).

### Использование:

Создайте `PostgreSqlManager` для управления соединением с базой данных:

```dart
var uri = "postgres://testdb:password@localhost:5432/testdb";
var dbManager = new PostgreSqlManager(uri, min: 1, max: 3);
```

Если вы используете redstone_mapper как плагин Redstone.dart, то можете передать `PostgreSqlManager` в `getMapperPlugin()`, что сделает соединение к базе данных доступным для каждого запроса:

```dart
import 'package:redstone/server.dart' as app;
import 'package:redstone_mapper/plugin.dart';
import 'package:redstone_mapper_pg/manager.dart';

main() {
  
  var uri = "postgres://testdb:password@localhost:5432/testdb";
  var dbManager = new PostgreSqlManager(uri, min: 1, max: 3);
  
  app.addPlugin(getMapperPlugin(dbManager));
  app.setupConsoleLog();
  app.start();
  
}

// redstone_mapper создаст атрибут "dbConn"
// для каждого запроса
@app.Route("/services/users/list")
listUsers(@app.Attr() PostgreSql dbConn) =>
   dbConn.innerConn.query("select * from users").toList();
   
// Если хотите, то можете создать геттер для доступа
// к соединению с базой для текущего запроса. Так вам не надо
// будет добавлять дополнительный параметр в каждый роут
PostgreSql get postgreSql => app.request.attributes.dbConn;

```

Объект `PostgreSql` предоставляет вспомогательные функции для кодирования и декодирования объектов:

```dart
import 'package:redstone/server.dart' as app;
import 'package:redstone_mapper/mapper.dart';
import 'package:redstone_mapper/plugin.dart';
import 'package:redstone_mapper_pg/manager.dart';

class User {
  
  @Field()
  int id;

  @Field()
  String username;

  @Field()
  String password;
  
}

PostgreSql get postgreSql => app.request.attributes.dbConn;

@app.Route("/services/users/list")
@Encode()
Future<List<User>> listUsers() => 
  // Получим докуметны из таблицы "users", декодируем результат
  // в List<User>
  postgreSql.query("select * from user", User);

@app.Route("/services/users/add", methods: const[app.POST])
Future addUser(@Decode() User user) => 
  // запишем пользователя в таблицу "users"
  postgreSql.execute("insert into users (name, password) "
                     "values (@username, @password)", user);

```

Класс `PostgreSql` не скрывает API пакета `postgresql`. Вы можете получить оригинальный объект соединения через свойство `PostgreSql.innderConn`.

Вы можете использовать `PostgreSqlService` для проведения операций над строками одной таблицы:

```dart

PostgreSqlService<User> userService = new PostgreSqlService<User>();

@app.Route("/services/users/list")
@Encode()
Future<List<User>> listUsers() => userService.query("select * from user"); 

@app.Route("/services/users/add", methods: const[app.POST])
Future addUser(@Decode() User user) => 
  postgreSql.execute("insert into users (name, password) "
                     "values (@username, @password)", user);

```

Также можно расширить `PostgreSqlService`:

```dart
@app.Group("/services/users")
Class UserService extends PostgreSqlService<User> {

  @app.Route("/list")
  @Encode()
  Future<List<User>> list() => query("select * from user");

  @app.Route("/add")
  Future add(@Decode() User user) =>
    execute("insert into users (name, password) "
            "values (@username, @password)", user);

}
```

`PostgreSqlService` по-умолчанию использует соединение, связанное с текущим запросом. Если вы не используете Redstone.dart, убедитесь, что создаете новый сервис через `PostgreSqlService.fromConnection()`.