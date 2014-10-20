---
layout: doc
menu_item: doc
title: Импорт библиотек
prev: Dependency-Injection
next: Plugin-API
---
Redstone.dart рекурсивно сканирует все библиотеки, импортируемые вашим главным скриптом. Пример:

- server.dart

```dart
import 'package:redstone/server.dart' as app;
// все слушатели, определенные в services.dart будут установлены
import 'package:myapp/services.dart';

main() {
  app.setupConsoleLog();
  app.start();
}
``` 
- services.dart

```dart
import 'package:redstone/server.dart' as app;

@app.Route("/user/find")
findUser() {
  ...
}
```

Но иногда нужно иметь возможность управлять установкой обработчиков из других библиотек. В этом случае вы можете использовать аннотацию `@Install`:

```dart
import 'package:redstone/server.dart' as app;
// все обработчики из services.dart, которые используют путь '/services' 
// будут установлены
@app.Install(urlPrefix: '/services')
import 'package:myapp/services.dart';

main() {
  app.setupConsoleLog();
  app.start();
}
``` 

Если библиотека определяет перехватчиков, то вы можете управлять очередью их выполнения:

```dart
import 'package:redstone/server.dart' as app;

@app.Install(chainIdx: 1)
import 'package:myapp/services.dart';

// этот перехватчик сработает первым
@app.Interceptor("/.+", chainIdx: 0)
interceptor() {
  print("interceptor 1");
  app.chain.next();
}

main() {
  app.setupConsoleLog();
  app.start();
}
``` 
- services.dart

```dart
import 'package:redstone/server.dart' as app;

@app.Interceptor("/.+", chainIdx: 0)
interceptor() {
  print("interceptor 2");
  app.chain.next();
}

@app.Interceptor("/.+", chainIdx: 1)
interceptor2() {
  print("interceptor 3");
  app.chain.next();
}
```

Если вы хотите импортировать библиотеку, но не использовать ее обработчики, то используйте аннотацию `@Ignore`:

```dart
import 'package:redstone/server.dart' as app;
// обработчики из services.dart не будут установлены
@app.Ignore()
import 'package:myapp/services.dart';

main() {
  app.setupConsoleLog();
  app.start();
}
``` 

Библиотека устанавливается только один раз. Поэтому, если вы импортируете одну библиотеку в нескольких файлах, то ее обработчики будут установлены только один раз:

```dart
import 'package:redstone/server.dart' as app;
import 'package:myapp/lib_a.dart';
import 'package:myapp/lib_b.dart';

main() {
  app.setupConsoleLog();
  app.start();
}
``` 
- lib_a.dart

```dart
library lib_a;
import 'package:redstone/server.dart' as app;
import 'package:myapp/lib_c.dart';

@app.Route(...)
serviceA() {
  ...
}
```
- lib_b.dart

```dart
library lib_b;
import 'package:redstone/server.dart' as app;
import 'package:myapp/lib_c.dart';

@app.Route(...)
serviceB() {
  ...
}
```
- lib_c.dart

```dart
// lib_c импортируется в файлах lib_a и lib_b, но ее обработчики будут
// установлены только один раз
library lib_c;
import 'package:redstone/server.dart' as app;

@app.Interceptor(...)
interceptor() {
  ...
}
```