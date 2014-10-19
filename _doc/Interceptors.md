---
layout: doc
menu_item: doc
title: Перехватчики
prev: Routes
next: Error-handlers
---
Используйте аннотацию `@Interceptor` для определения перехватчика:

```dart
@app.Interceptor(r'/.*')
handleResponseHeader() {
  if (app.request.method == "OPTIONS") {
    // изменим текущий ответ и прервем цепочку
    app.response = new shelf.Response.ok(null, headers: _createCorsHeader());
    app.chain.interrupt();
  } else {
    // продолжим выполнение цепочки и изменим ответ
    app.chain.next(() => app.response.change(headers: _createCorsHeader()));
  }
}

_createCorsHeader() => {"Access-Control-Allow-Origin": "*"};
```

## Объект цепочки (chain)

Каждый запорс представляет из себя цепочку, в которой 0 или более перехватчиков и один маршрут. Перехватчики - это структура, которая позволяет применить определенное поведение к ряду ссылок. Например, вы можете использовать перехватчик для ограничения доступа, или управления ресурсом:

```dart
@app.Interceptor(r'/admin/.*')
adminFilter() {
  if (app.request.session["username"] != null) {
    app.chain.next();
  } else {
    app.chain.interrupt(statusCode: HttpStatus.UNAUTHORIZED);
    // или app.redirect("/login.html");
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

Когда запрос принят, фреймворк запустит всех перехватчиков, которые совпадают с URL, а затем поищет подходящий маршрут. Если маршрут будет найден, то связанная с ним функция или метод выполнится.

Каждый перехватчик должен вызвать методы `chain.next()` или `chane.interrupt()`, иначе запрос не пройдет дальше. Метод `chane.next()`, может принять `callback`, который выполнится после обработки запроса. Все обратные вызовы будут выполнены в обратном их созданию порядке. Если `callback` вернет `Future`, то следующий обратный вызов выполнится только после выполнения `Future`.

Посмотрите следующий пример:

```dart
import 'package:redstone/server.dart' as app;
import 'package:shelf/shelf.dart' as shelf;

@app.Route("/")
helloWorld() => "target\n";

@app.Interceptor(r'/.*', chainIdx: 0)
interceptor1() {
  app.chain.next(() {
    return app.response.readAsString().then((resp) =>
        new shelf.Response.ok(
          "interceptor 1 - before target\n$resp|interceptor 1 - after target\n"));
  });
}

@app.Interceptor(r'/.*', chainIdx: 1)
interceptor2() {
  app.chain.next(() {
    return app.response.readAsString().then((resp) =>
        new shelf.Response.ok(
          "interceptor 2 - before target\n$resp|interceptor 2 - after target\n"));
  });
}

main() {

  app.setupConsoleLog();
  app.start();
  
}
```

Когда вы зайдете сюда: http://127.0.0.1:8080/; то увидите:

```
interceptor 1 - before target
interceptor 2 - before target
target
interceptor 2 - after target
interceptor 1 - after target
```

Также можно проверить, была ли брошена ошибка (если зарегистрирован обработчик ошибок, то он отработает перед вызовом `callback`):

```dart
@app.Interceptor(r'/.*')
interceptor() {
  app.chain.next(() {
    if (app.chain.error != null) {
      ...
    }
  });
}
```

## Тело запроса

По-умолчанию Redstone.dart не парсит тело запроса до тех пор, пока не отработают все перехватчики. Если ваш перехватчик должен поработать с телом запроса, то установите `parseRequestBody = true`. Пример:

```dart
@app.Interceptor(r'/service/.+', parseRequestBody: true)
verifyRequest() {
  // если parseRequestBody не равен true, то request.body будет null
  print(app.request.body);
  app.chain.next();
}

```

## Контроль очереди выполнения

Вы можете контролировать, в каком порядке перехватчики будут вызваны, с помощью параметра `chainIdx`:

```dart

@app.Interceptor("/.+", chainIdx: 0)
interceptor() {
  print("перехватчик 2");
  app.chain.next();
}

@app.Interceptor("/.+", chainIdx: 1)
interceptor2() {
  print("перехватчик 3");
  app.chain.next();
}
```