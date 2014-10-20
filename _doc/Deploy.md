---
layout: doc
menu_item: doc
title: Деплой
prev: Server-Configuration
next: redstone_mapper
---

Простеший путь сборки вашего приложения - это использование [Grinder](http://pub.dartlang.org/packages/grinder). Redstone.dart предоставляет простую задачу для копирования файлов сервера в директорию сборки. Эту задачу вы можете использовать для создания скрипта сборки.

**Примечание:** с версии 0.5.18 Redstone.dart использует новую версию Grinder (0.6.x), которая включает некоторые несовместимые изменения. Также, в новой версии появилась возможность вызова скрипта сборки через команду `pub run`. 

## Создание скрипта сборки

Создайте файл `grind.dart` в папке `tool`.

### Redstone.dart 0.5.18 и выше

```dart
import 'package:grinder/grinder.dart';
import 'package:redstone/tasks.dart';

main(List<String> args) {
  task('build', Pub.build);
  task('deploy_server', deployServer, ['build']);
  task('all', null, ['build', 'deploy_server']);

  startGrinder(args);
}
```

### Redstone.dart v0.5.17 и ниже

```dart
import 'package:grinder/grinder.dart';
import 'package:grinder/grinder_utils.dart';
import 'package:redstone/tasks.dart';

main(List<String> args) {
  defineTask('build', taskFunction: (GrinderContext ctx) => new PubTools().build(ctx));
  defineTask('deploy_server', taskFunction: deployServer, depends: ['build']);
  defineTask('all', depends: ['build', 'deploy_server']);
  
  startGrinder(args);
}
```

## Запуск скрипта сборки из Dart Editor

Для выполнения `grind.dart` из Dart Editor вам нужно создать конфигурацию окружения командной строки со следующими параметрами:

Параметр          | Значение
------------------|----------
Dart Script       | tool/grind.dart
Working directory | (путь до папки проекта)
Script arguments  | all

## Запуск скрипта сборки из командной строки

### Redstone.dart v0.5.18 и выше

Используйте команду `pub run` для запуска скрипта сборки (в `pubspec.yaml` нужно включить зависимость от пакета `grinder`):

```
$ pub run grinder:grind all
```

Если вы установили Grinder через команду [pub global](https://www.dartlang.org/tools/pub/cmd/pub-global.html), то можете вызвать `grind` напрямую:

```
$ grind all
```

### Redstone.dart v0.5.17 и ниже

Для запуска `grind.dart` из командной строки, вы должны установить переменную окружения `DART_SDK`:

```
$ export DART_SDK=(path to dart-sdk)
$ dart tool/grind.dart all
```

##DartVoid

Если вы хотите загрузить ваше приложение на [DartVoid](http://www.dartvoid.com/), то вам не нужен скрипт сборки (DartVoid разберется с этим сам), но вы должны разделить ваш проект на два: `server` и `client`.

DartVoit предоставляет набор шаблонов приложений, построенных на разных фреймворках, включая Redstone.dart, которые вы можете использовать для создания вашего приложения:

* [Hello World](https://github.com/DartTemplates/Redstone-Hello)
* [Guestbook](https://github.com/DartTemplates/Redstone-Guestbook)
* [Todo List](https://github.com/DartTemplates/Redstone-Angular-Todo)

##Heroku

Вы можете достаточно просто загружать ваши Dart приложения на [Heroku](https://www.heroku.com/) c помощью стека [cedar-14](https://blog.heroku.com/archives/2014/8/19/cedar-14-public-beta) и [Dart buildpack](https://github.com/igrigorik/heroku-buildpack-dart). Вы можете увидеть больше примеров [здесь](https://github.com/luizmineo/io_2014_contacts_demo).