---
layout: doc
menu_item: doc
title: Shelf Middlewares
prev: Plugin-API
next: Unit-test
---
С версии 0.5 Redstone.dart базируется на фреймворке [Shelf](http://pub.dartlang.org/packages/shelf). Это означает, что вы можете использовать Shelf-middleware или его обработчики в Вашем приложении:

```dart

main() {
  // Middleware, зарегистрированные с помощью функции addShelfMiddleware() 
  // будут вызваны перед перехватчиками и маршрутами
  app.addShelfMiddleware(...);
  app.addShelfMiddleware(...);
  
  // Обработчики, зарегистрированные с помощью функции setShelfHandler()
  // будут вызваны после отработки всех перехватчиков и когда для указанного
  // URL не будет найден маршрут
  app.setShelfHandler(...);

  app.setupConsoleLog();
  app.start();

}

```

Например, вы можете использовать [shelf_static](http://pub.dartlang.org/packages/shelf_static) для работы со статическими файлами:

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