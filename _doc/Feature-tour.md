---
layout: doc
menu_item: doc
title: Обзор возможностей
prev: Installation
next: Routes
---
## Маршруты

Для привязки URL к функции используйте нотацию `@Route`

```dart
@app.Route("/")
helloWorld() => "Hello, World!";
```

Redstone.dart сериализует результат функции автоматически. Если функция возвратит `List` или `Map`, то клиент получит JSON-объект:

```dart
@app.Route("/user/find/:id")
getUser(String id) => {"name": "User", "login": "user"};
```

Если функция зависит от выполнения асинхронной операции, можно вернуть `Future`

```dart
@app.Route("/service")
service() => doSomeAsyncOperation().then((_) => {"success": true});
```

Можно легко получать сегменты пути и параметры запроса

```dart
@app.Route("/user/find/:type")
findUsers(String type, @app.QueryParam() String name) {
  ...
}
```

Также можно получить тело запроса

```dart
@app.Route("/user/add", methods: const [app.POST])
addUser(@app.Body(app.JSON) Map user) {
  ...
}
```

Есть возможность получения доступа к текущему объекту запроса

```dart
@app.Route("/service", methods: const [app.GET, app.POST])
service() {
  if (app.request.method == app.GET) {
    ...
  } else if (app.request.method == app.POST) {
    
    if (app.request.bodyType == app.JSON) {
      var json = app.request.body;
      ...
    } else {
      ...
    }
  }
};
```

## Перехватчики

Перехватчики полезны когда нужно применить какое-то поведение к группе ссылок (функций или статическому контенту). Например. вы можете создать перехватчика для ограничения доступа или для создания ресурса (в примере это будет подключение к базе данных).

```dart
@app.Interceptor(r'/admin/.*')
adminFilter() {
  if (app.request.session["username"] != null) {
    app.chain.next();
  } else {
    app.chain.interrupt(statusCode: HttpStatus.UNAUTHORIZED);
    //или app.redirect("/login.html");
  }
}
```

```dart
@app.Interceptor(r'/services/.+')
dbConnInterceptor() {
  var conn = new DbConn();
  app.request.attributes["dbConn"] = conn;
  app.chain.next(() => conn.close());
}

@app.Route('/services/find')
find(@app.Attr() dbConn) {
  ...
}
```

## Слушатели ошибок

Используйте аннотацию `@ErrorHandler` для регистрации слушателя ошибок

```dart
@app.ErrorHandler(404)
handleNotFoundError() => app.redirect("/error/not_found.html");
```

```dart
@app.ErrorHandler(500)
handleServerError() {
  print(app.chain.error);
  return new shelf.Response.internalServerError(body: "Server Error.");
}
```

## Группы

Вы можете использовать классы для группировки маршрутов, перехватчиков и слушателей ошибок

```dart
@Group("/user")
class UserService {
  
  @app.Route("/find")
  findUser(@app.QueryParam("n") String name,
           @app.QueryParam("c") String city) {
    ...
  }

  @app.Route("/add", methods: const [app.POST])
  addUser(@app.Body(app.JSON) Map json) {
    ...
  }
}
```

## Dependency Injection

Зарегистрируйте один или несколько модулей перед вызовом `app.start()`

```dart
import 'package:redstone/server.dart' as app;
import 'package:di/di.dart';

main() {

  app.addModule(new Module()
       ..bind(ClassA)
       ..bind(ClassB));
  
  app.setupConsoleLog();
  app.start();

}

```

Маршруты, перехватчики, слушатели ошибок и группы могут подключать зависимости

```dart
@app.Route('/service')
service(@app.Inject() ClassA objA) {
 ...
}
```

```dart
@app.Interceptor(r'/services/.+')
interceptor(ClassA objA, ClassB objB) {
  ...
}
```

```dart
@app.ErrorHandler(404)
notFound(ClassB objB) {
  ...
}
```

```dart
@app.Group('/group')
class Group {

  ClassA objA;
  
  Group(ClassA this.objA);
  
  @app.Route('/service')
  service() {
    ...
  }

}
```

### Тестирование

Вы легко можете создавать фиктивные запросы для проверки сервера

```dart
library services;

import 'package:redstone/server.dart' as app;

@app.Route("/user/:username")
helloUser(String username) => "hello, $username";
```

```dart
import 'package:unittest/unittest.dart';

import 'package:redstone/server.dart' as app;
import 'package:redstone/mocks.dart';

import 'package:your_package_name/services.dart';

main() {

  // загрузим слушателей из библиотеки 'services'
  setUp(() => app.setUp([#services]));
  
  // удалим все загруженные слушатели
  tearDown(() => app.tearDown());
  
  test("hello service", () {
    // создадим запрос
    var req = new MockRequest("/user/luiz");
    // отправим запрос
    return app.dispatch(req).then((resp) {
      // проверим ответ
      expect(resp.statusCode, equals(200));
      expect(resp.mockContent, equals("hello, luiz"));
    });
  })
  
}
```