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

### Валидация данных

Класс `Validator` предоставляет простой и гибкий способ для создания набора правил валидации.

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

Для валидации объектов вы можете передать нужный класс в конструктор. Также вы должны проставить аннотацию `@Field` у каждого свойства, значение которого будет проверяться.

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

Другой способ - это описание правил валидации прямо у свойств:

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

Также вы можете наследоваться от класса `Schema`, который предоставляет `Validator`.

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

redstone_mapper предоставляет следующие базовые правила, которые вы можете использовать для построения валидации:

* `NotEmpty`:
    * Если значение строка, то проверит ее на `null`, пустая ли она или содержит только пробельные символы.
    * Если значение `Iterable`, то проверит, не является ли оно `null` или пустым.
    * Другие значения просто проверит на то, не являются ли они `null`.
* `Range`:
    * Если значение число, то проверит, попадает ли оно в указанный диапазон
    * Если значение строка или `Iterable`, то проверит, попадает ли их длина в указанный диапазон.
* `Matches`:
    * Только для строк. Проверяет, соответствует ли строка регулярному выражению.
* `OnlyNumbers`:
    * Только для строк. Проверят, содержит ли строка только цифровые символы.

Вы можете очень просто создавать новые правила, расширив класс `ValidationRule`.

### Настройка

Для установки redstone_mapper в качестве плагина Redstone.dart, вы должны импортировать `plugin.dart` и вызвать `getMapperPlugin()`:

```dart

import 'package:redstone/server.dart' as app;
import 'package:redstone_mapper/plugin.dart';

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

```

Если `getMapperPlugin()` принимает объект `DatabaseManager`, то плагин будет управлять соединением с базой за вас. Для более подробной информации, посмотрите расширения этого плагина, например [redstone_mapper_pg](https://github.com/luizmineo/redstone_mapper_pg) или [redstone_mapper_mongo](https://github.com/luizmineo/redstone_mapper_mongo).

Для использование с другими серверными фреймворками, или для работы на клиентской стороне, вы можете просто импортировать `mapper_factory.dart` и вызвать `bootstrapMapper()` в функции `main()`:

```dart

import 'package:redstone/server.dart' as app;
import 'package:redstone_mapper/mapper_factory.dart';

main() {

  bootstrapMapper();
  ...
}

```

Для кодирование и декодирования объектов вы можете использовать top-level функции `encode()` и `decode()`, которые определены в файле `mapper.dart`:

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

Когда вы используете redstone_mapper на серверной стороне, убедитесь, что прописали трансформер redstone_mapper в pubspec.yaml - благодаря этому dart2js не будет генерировать большой javascript-файл:

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