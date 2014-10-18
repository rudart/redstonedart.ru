---
layout: doc
menu_item: doc
title: Введение
next: Installation
---
Redstone.dart это написанный на [Dart](https://www.dartlang.org/) серверный микрофреймворк, принцип работы которого основывается на использовании метаданных.

#### Как это работает?

Redstone.dart позволяет легко давать доступ к функциям и классам через веб-интерфейс. Нужно просто добавить аннотацию.

```dart

import 'package:redstone/server.dart' as app;

@app.Route("/")
helloWorld() => "Hello, World!";

main() {
  app.setupConsoleLog();
  app.start();
}
``` 

Выглядит знакомо? Redstone.dart взял многие идеи и концепции из микрофреймворка [Flask](http://flask.pocoo.org/), написанного на Python.
