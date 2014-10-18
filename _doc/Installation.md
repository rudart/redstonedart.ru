---
layout: doc
menu_item: doc
title: Установка
prev: index
next: Feature-tour
---

Redstone.dart доступен как пакет [pub](http://pub.dartlang.org/). Поэтому, для установки Redstone его нужно просто добавить в зависимости приложения.

* Создайте новый пакет Dart ([вручную](http://pub.dartlang.org/doc/) или с помощью Dart Editor)
* Добавьте Redstone.dart как зависимость в `pubspec.yaml'

```
name: my_app
 dependencies:
   redstone: any
```
- Выполните команду `pub get` для обновления зависимостей
- Создайте директорию `bin`
- Создайте файл `server.dart` в папке `bin`

```dart

import 'package:redstone/server.dart' as app;

@app.Route("/")
helloWorld() => "Hello, World!";

main() {
  app.setupConsoleLog();
  app.start();
}

```

- Для запуска сервера создайте конфигурацию в Dart Editor, или используйте команду:

```
$ dart bin/server.dart
INFO: 2014-02-24 13:16:19.086: Configured target for / [GET] : .helloWorld
INFO: 2014-02-24 13:16:19.121: Running on 0.0.0.0:8080
```

- Теперь зайдите сюда http://127.0.0.1:8080/. Вы должны будете увидеть надипись "Hello, World!".
