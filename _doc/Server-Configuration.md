---
layout: doc
menu_item: doc
title: Конфигурация сервера
prev: Unit-test
next: Deploy
---

Если вызвать метод `start()` без параметров, то сервер будет запущен со стандартными параметрами:

Параметр       | Стандартное значение
---------------|---------------------
host           | "0.0.0.0"
port           | 8080
protocol       | http

## Безопасное соединение (https)

Чтобы запустить https-сервер, вы должны указать необязательный параметр `secureOptions` в вызове функции `start()`.

Параметр `secureOptions` имеет тип `Map<Symbol, dynamic>`. Этот параметр будет передан методу [`HttpServer.bindSecure()`](https://api.dartlang.org/apidocs/channels/stable/dartdoc-viewer/dart-io.HttpServer#id_bindSecure):

```dart
import 'package:redstone/server.dart' as app;

main() {
  app.setupConsoleLog();
  app.start(secureOptions: {#certificateName: "CN=RedStone"});
}
```

Рабочий пример можно посмотреть здесь: [`https.dart`](https://github.com/luizmineo/redstone.dart/blob/master/example/https.dart).

## Статичные файлы

Если вам нужно обрабатывать статичные файлы, то можете использовать пакет [shelf_static](http://pub.dartlang.org/packages/shelf_static):

```dart
import 'package:redstone/server.dart' as app;
import 'package:shelf_static/shelf_static.dart';

main() {
  app.setShelfHandler(createStaticHandler("../web", 
                                          defaultDocument: "index.html", 
                                          serveFilesOutsidePath: true));
  app.setupConsoleLog();
  app.start();
}
```
## Логи

Redstone.dart предоставляет вспомогательный метод для установки обработчика логов, который выводит сообщения на консоль:

```dart
app.setupConsoleLog();
```

По-умолчанию уровень логов установлен в `INFO`, который регистрирует процесс запуска и ошибки. Если вам нужно видеть все сообщения, то можете установить уровень в `ALL`:

```dart
import 'package:logging/logging.dart';


main() {
  app.setupConsoleLog(Level.ALL);
  ...
}
```

Если вы хотите выводить сообщения в другое место (например, в файл), то можно определить свой обработчик логов:

```dart
Logger.root.level = Level.ALL;
Logger.root.onRecord.listen((LogRecord rec) {
  ...
});
```