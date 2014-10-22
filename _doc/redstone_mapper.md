---
layout: doc
menu_item: doc
title: redstone_mapper
prev: Deploy
next: redstone_mapper_mongo
---
[redstone_mapper](http://pub.dartlang.org/packages/redstone_mapper) - это набор утилит для выполнения наиболее частых задач в веб-приложениях:

* Кодирование/декодирование объектов в/из JSON
* Валидация данных
* Управление соединением с базой данных
* Сохранение и извлечение объектов из/в БД

Кодирование/декодирование объектов и валидация данных также может быть использована на клиенте. **redstone_mapper** предоставляет трансформер pub, который предотвращает генерацию раздутых javascript-файлов dart2js.

Пример: Использование redstone_mapper с Redstone.dart

```dart

import 'package:redstone/server.dart' as app;
import 'package:redstone_mapper/mapper.dart';
import 'package:redstone_mapper/plugin.dart';

main() {

  // Когда redstone_mapper используется как плагин Redstone.dart,
  // тогда мы можем использовать аннотации @Decode и @Encode
  app.addPlugin(getMapperPlugin());
  
  app.setupConsoleLog();
  app.start();
}

class User {

  // Аннотация @Field используется для указания
  // поля, которое может быть сериализовано
  @Field()
  String username;

  @Field()
  String password;
  
}

// Аннотация @Decode говорит, что значение параметра запроса
// должно быть декодировано. В большинстве случаев это означает,
// что тело запроса должно содержать JSON
@app.Route('/services/users/add', methods: const[app.POST])
addUser(@Decode() User user) {
  ...
}

// Аннотация @Encode говорит, что ответ маршрута
// должен быть закодирован в JSON
@app.Route('/services/users/list')
@Encode()
List<User> listUsers() {
  ...
}

```

### Аннотация @Field

Чтобы верно кодировать и декодировать объект, класс этого объекта должен иметь у каждого сериализуемого свойства аннотацию `@Field`.

```dart

class User {

  @Field()
  String username;

  @Field()
  String password;
  
}

```

Обязательно нужно указать тип свойства, т.к. основываясь на этой информации будет происходить кодирование и декодирование. Если тип поля `List` или `Map`, то убедитесь, что этот тип указан. Пример:

```dart

class User {
  
  @Field()
  String name;
 
  @Field()
  List<Address> adresses;

}

class Address {

  @Field()
  String description;

  @Field()
  int number;
  
}

```

Не рекомендуется использовать другие классы в качестве типов сериализуемых свойств, т.к. тогда не гарантируется правильное кодирование и декодирование объекта.

Также можно использовать аннотацию `@Field` с геттерами и сеттерами:

```dart
class User {

  String _name;
  
  @Field()
  String get name() => _name;

  @Field()
  set name(String value) => _name = value;
  
}

```

When encoding or decoding an object to JSON, you can use the `view` parameter
to map a class member to its corresponding JSON field:

```dart

class User {
  
  @Field(view: "user_name")
  String name;
  
  @Field()
  String pass;
  
}

```

Если вы записываете объект в базу данных, или получаете его из нее, то можете использовать параметр `model` для связки свойства класса с полем в базе данных:

```dart

class User {
  
  //JSON: 'user_name'  DATABASE: 'USERNAME'
  @Field(view: "user_name", model: "USERNAME")
  String name;
  
  //JSON: 'pass'  DATABASE: 'PASSWORD'
  @Field(model: "PASSWORD")
  String pass;
}

```

Каждый класс, который может быть кодирован/декодирован должен иметь стандартный конструктор без обязательных параметров.

### Data validation

The `Validator` class provides a simple and flexible way to build a set of validation rules.

```dart
  var userValidator = new Validator()
                      ..add("username", const NotEmpty())
                      ..add("password", const Range(min: 6. required: true));

  ...
  Map user = {"username": "user", "password": "pass"};
  ValidationError err = userValidator.execute(user);
  if (err != null) {
    ...
  }
```

To validate objects, you must provide the target class to the constructor. Also,
you must annotate with `@Field` all members that can be validated.

```dart
class User {
     
  @Field()
  String username;
 
  @Field()
  String password;

}

var userValidator = new Validator(User)
                    ..add("username", const NotEmpty())
                    ..add("password", const Range(min: 6. required: true));

...
var user = new User()
            ..username = "user"
            ..password = "pass";
              
ValidationError err = userValidator.execute(user);
if (err != null) {
  ...
}
```

Alternatively, you can set the rules directly in the class. 

```dart
class User {
   
  @Field()
  @NotEmpty()
  String username;

  @Field()
  @Range(min: 6, required: true)
  String password;

}

var userValidator = new Validator(User, true);
```

You can also inherit from the `Schema` class, which will provide a `Validator`
for you.

```dart
class User extends Schema {
 
  @Field()
  @NotEmpty()
  String username;

  @Field()
  @Range(min: 6, required: true)
  String password;

}

...
User user = new User()
            ..username = "user"
            ..password = "pass";
            
var err = user.validate();
if (err != null) {
  ...
}
```

redstone_mapper already provides the following basic rules, that you can use
to build a `Validator`: 

* `NotEmpty`:
    * If the value is a String, verify if it isn't null, empty, or contains only spaces.
    * If the value is an Iterable, verify if isn't null or empty.
    * For other values, verify if it isn't null.
* `Range`:
    * If the value is numeric, verify if it's within the specified range.
    * If the value is a String or an Iterable, verify if its length is within the specified range.
* `Matches`:
    * For strings only: verify if the value matches the specified regex.
* `OnlyNumbers`:
    * For strings only: verify if the value contains only digit characters.

You can easily build new rules by just inheriting from the `ValidationRule` class.

### Configuration

To install redstone_mapper as a Redstone.dart plugin, you just have to import `plugin.dart` and
call `getMapperPlugin()`:

```dart

import 'package:redstone/server.dart' as app;
import 'package:redstone_mapper/plugin.dart';

import 'package:redstone/server.dart' as app;
import 'package:redstone_mapper/mapper.dart';
import 'package:redstone_mapper/plugin.dart';

main() {

  //When using redstone_mapper as a Redstone.dart plugin,
  //you can use the @Decode and @Encode annotations.
  app.addPlugin(getMapperPlugin());
  
  app.setupConsoleLog();
  app.start();
}

```

Also, if `getMapperPlugin()` receives an instance of `DatabaseManager`, then the plugin will manage
the database connections for you. For more information, see one of the redstone_mapper extensions, such as
[redstone_mapper_pg](https://github.com/luizmineo/redstone_mapper_pg) or 
[redstone_mapper_mongo](https://github.com/luizmineo/redstone_mapper_mongo).

To use with other server-side frameworks, or on the client side, you just have to import `mapper_factory.dart`
and call `bootstrapMapper()` from the `main()` function:

```dart

import 'package:redstone/server.dart' as app;
import 'package:redstone_mapper/mapper_factory.dart';

main() {

  bootstrapMapper();
  ...
}

```

To encode and decode objects, you can use the `encode()` and `decode()` top level function from `mapper.dart`:

```dart

import 'dart:convert';
import 'package:redstone_mapper/mapper.dart';

class User {
     
  @Field()
  String username;
 
  @Field()
  String password;

}

User user = new User()
            ..username = "user"
            ..password = "pass";
            
String userJson = JSON.encode(encode(user));

```

When using on the client side, be sure to set redstone_mapper's transformer to your pubspec.yaml
file, so dart2js won't generate a bloated javascript file:

```
name: my_app
version: 0.1.0
dependencies:
  redstone: any
  redstone_mapper: any
transformers:
- redstone_mapper

```

### Integration with Polymer

Polymer applications usually doesn't have an entry-point (a dart script with the `main` function), so
you have to provide one. Also, the entry-point has to import all libraries that contains encodable classes, 
so the transformer will be able to map them. You can see a working example which uses 
polymer and redstone_mapper [here](https://github.com/luizmineo/io_2014_contacts_demo).

### Database integration

redstone_mapper provides integration with database drivers through extensions. Currently, the following extensions are available:

* [redstone_mapper_mongo](https://github.com/luizmineo/redstone_mapper_mongo): MongoDB extension for redstone_mapper.
* [redstone_mapper_pg](https://github.com/luizmineo/redstone_mapper_pg): PostgreSQL extension for redstone_mapper.

Note that redstone_mapper doesn't aim to be a full ORM/ODM framework. It just provides some helper functions to easily 
encode and decode objects to the database. It won't generate database queries, neither hide the default driver API from you.
That means you can use the redstone_mapper functions only when it's useful for you, and ignore it when it's just an extra overhead. 

#### What about other databases?

Dart already has support for several databases, including: MongoDb, Redis, CouchDb, MySql, PostgreSql, and so on. I'll try to provide new extensions over
time, but if you are interested, you can help me on this task.

Building a redstone_mapper extension is really easy, and you can start by taking a look at the source code of [redstone_mapper_pg](https://github.com/luizmineo/redstone_mapper_pg) and [redstone_mapper_mongo](https://github.com/luizmineo/redstone_mapper_mongo).
If you are willing to build a externsion, please let me know :)