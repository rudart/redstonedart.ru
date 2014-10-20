---
layout: doc
menu_item: doc
title: Тестирование
prev: Shelf-Middlewares
next: Server-Configuration
---
Чтобы создать тест, вы должны:

* Вызвать `setUp()` для загрузки обработчиков
* Создать `MockRequest`
* Вызвать `dispatch()` для выполнения запроса
* Изучить ответ
* Вызвать `tearDown()` для удаления обработчиков

Пример:

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

  // загружаем обработчики из библитеки 'services'
  setUp(() => app.setUp([#services]));
  
  // Удалим все обработчики
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

## Примеры MockRequest

* GET-запрос: /service

```dart
var req = new MockRequest("/service");
```

* GET-запрос: /service?param=value

```dart
var req = new MockRequest("/service", queryParams: {"param": "value"});
```

* POST-запрос: /service, JSON data

```dart
var req = new MockRequest("/service", method: app.POST, bodyType: app.JSON, body: {
  "key1": "value1",
  "key2": "value2"
});
```

* POST-запрос: /service, FORM data

```dart
var req = new MockRequest("/service", method: app.POST, bodyType: app.FORM, body: {
  "key1": "value1",
  "key2": "value2"
});
```

* POST-запрос: /service, FORM data, multipart-запрос

```dart
import "dart:convert";
import "dart:io";

...

var file = new app.HttpBodyFileUpload(ContentType.parse("text/plain"), 
                                      "test.txt", 
                                      UTF8.encode("test"));

var req = new MockRequest("/service", method: app.POST, bodyType: app.FORM, body: {
  "key1": "value1",
  "file": file
});
```

* Устанавливаем сессию

```dart
var req = new MockRequest("/service", 
  session: new MockHttpSession("session_id", {"user": "username"}));
```
