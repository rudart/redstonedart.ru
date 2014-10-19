---
layout: doc
menu_item: doc
title: Маршруты
prev: Feature-tour
next: Interceptors
---
Аннотация `@Route` используется для привязки URL к функции или методу:

```dart
@app.Route("/")
helloWorld() => "Hello, World!";
```

Значение, которое вернет функция будет сериализовано в соответствии с его типом. Например, если результатом функции будет строка, то клиент получит ответ в *text/plane*.

Тип результата | Тип ответа
---------------|---------------
String         | text/plain
Map or List    | application/json
File           | (MimeType файла)

Если функция вернет `Future`, то фреймворк дождется его выполнения.

```dart
@app.Route("/")
helloWorld() => new Future(() => "Hello, World!");
```

Если вам нужно ответить на запрос кодом состояния, отличным от 200, то вы можете вернуть или сбросить `ErrorResponse`:

```dart
@app.Route("/user/:id")
getUser(int id) {
  if (id <= 0) {
    throw new app.ErrorResponse(400, {"error": "invalid id"});
  }
  ...
}
```
Также вы можете построить ответ, используя [Shelf](http://pub.dartlang.org/packages/shelf).

```dart
import 'package:redstone/server.dart' as app;
import 'package:shelf/shelf.dart' as shelf;

@app.Route("/")
helloWorld() => new shelf.Response.ok("Hello, World!");
```

Остальные типы Redstone.dart сконвертирует в строку и отправит ответ в *text/plain*.

Есть возможность явно указать тип ответа:

```dart
@app.Route("/", responseType: "text/xml")
getXml() => "<root><node>text</node></root>";
```

## Параметры

### Сегмент пути

Можно связывать параметр с сегментом пути:

```dart
@app.Route("/user/:username")
helloUser(String username) => "hello $username";
```

Аргумент не обязательно должен быть строкой. Например, если аргумент - целое число, то фреймворк попробует автоматически конвертировать значение в число (если конвертировать значение не получится, то клиенту будет сброшена ошибка 400).

```dart
@app.Route("/user/:username/:addressId")
getAddress(String username, int addressId) {
  ...
};
```

Поддерживаемые типы: `int`, `double` и `bool`.

### Параметры запроса

Для доступа к параметрам запроса используйте аннотацию `@QueryParam`

```dart
@app.Route("/user")
getUser(@app.QueryParam("id") int userId) {
  ...
};
```

Как и в параметрах, которые передаются как часть пути, параметр запроса не обязательно должен быть строкой.

### Тело запроса

Вы можете получить доступ к телу запроса (форма, json или простой текст)

```dart
@app.Route("/adduser", methods: const [app.POST])
addUser(@app.Body(app.JSON) Map json) {
  ...
};
```

```dart
@app.Route("/adduser", methods: const [app.POST])
addUser(@app.Body(app.FORM) Map form) {
  ...
};
```

Для json и форм есть возможность получить тело запроса как объект `QueryMap`, который позволит получить доступ к элементам через точку.

```dart
@app.Route("/adduser", methods: const [app.POST])
addUser(@app.Body(app.JSON) QueryMap json) {
  var name = json.name;
  ...
};
```

## HTTP-методы

По умолчанию, маршрут реагирует только на GET-запрос. Вы можете изменить это с помощью параметра `methods`:

```dart
@app.Route("/user/:username", methods: const [app.GET, app.POST])
helloUser(String username) => "hello $username";
```

Также можно сделать так, что по одному пути будут отрабатывать разные методы, в зависимоти от HTTP-метода:

```dart
@app.Route("/user", methods: const [app.GET])
getUser() {
 ...
};

@app.Route("/user", methods: const [app.POST])
postUser(@app.Body(app.JSON) Map user) {
 ...
};
```

## Multipart-запросы (загрузка файлов)

По-умолчанию, Redstone.dart сбросит любой multipart-запрос. Если ваш метод должен принять multipart-запрос, то укажите `Route.allowMultipartRequest = true`. Пример:

```dart
@app.Route("/adduser", methods: const [app.POST], allowMultipartRequest: true)
addUser(@app.Body(app.FORM) Map form) {
  var file = form["file"];
  print(file.filename);
  print(file.contentType);
  print(file.content);
  ...
};
```

## Совпадение подпутей

Если установить `Route.matchSubPaths = true`, то маршрут будет обрабатывать запросы, пути которых начинаются с указанного паттерна URL. Пример:

```dart
@app.Route('/path', matchSubPaths: true)
service() {
 ...
}

@app.Route('/path/subpath')
serviceB() {
 ...
}
```

Если принят запрос на `/path/subpath`, то будет выполнена функция `serviceB`; но если принят запрос `/path/another_path`, то будет выполнена функция `service`.

Также вы можете связать подпуть запроса с параметром, добавив в конце шаблона URL символ `*`. Пример:

```dart
@app.Route('/service/:path*', matchSubPaths: true)
service(String path) {
 ...
}
```

## Объект запроса

Вы можете использовать глобальный объект `request` для доступа к информации о запросе и к его контенту:

```dart
@app.Route("/user", methods: const [app.GET, app.POST])
user() {
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

Каждый запрос связан со своей собственной [зоной (Zone)](https://www.dartlang.org/articles/zones/), поэтому можно безопасно получить доступ к объекту запроса в асинхронных операциях.

## Объект ответа

Иногда нужно вручную создать HTTP-ответ, или проверить и изменить ответ, созданный другим обработчиком (маршрутом, перехватчиком или слушателем ошибок). В таких случаях можно обратиться к объекту `response`, который указывает на созданный ответ для данного запроса. Пример:

```dart
import 'package:redstone/server.dart' as app;

@app.Interceptor(r'/.*')
interceptor() {
  app.chain.next(() {
    app.response = app.response.change(headers: {
      "Access-Control-Allow-Origin": "*"
    });
  });
}
```

Если вы строите ответ внутри **chain callback**, **маршрута** или **обработчика ошибок**, то вы можете просто вернуть объект `request`, без присвоения:

```dart
import 'package:redstone/server.dart' as app;

@app.Interceptor(r'/.*')
interceptor() {
  app.chain.next(() {
    return app.response.change(headers: {
      "Access-Control-Allow-Origin": "*"
    });
  });
}
```