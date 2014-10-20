---
layout: doc
menu_item: doc
title: Внедрение зависимостей
prev: Groups
next: Importing-libraries
---
Для реализации внедрения зависимостей Redstone.dart использует [пакет di](http://pub.dartlang.org/packages/di).

Регистрация модуля осуществляется с помощью метода `addModule()`:

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

В методы, у которых есть аннотация `@Route` можно внедрять зависимости с помощью аннотации `@Inject`:

```dart
@app.Route('/service')
service(@app.Inject() ClassA objA) {
 ...
}
```

Группы могут получать зависимости в конструкторе:

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

Перехватчики и слушатели ошибок тоже могут использовать зависимости:

```dart
@app.Interceptor(r'/services/.+')
interceptor(ClassA objA, ClassB objB) {
  ...
}


@app.ErrorHandler(404)
notFound(ClassB objB) {
  ...
}
```