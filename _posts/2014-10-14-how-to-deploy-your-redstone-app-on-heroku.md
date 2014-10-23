---
layout: post
title: "Как разместить приложение Redstone.dart на Heroku"
author: Luiz Mineo
---

Сегодня я немного поиграл с [cedar-14](https://blog.heroku.com/archives/2014/8/19/cedar-14-public-beta) и [Dart buildpack](https://github.com/igrigorik/heroku-buildpack-dart) и был удивлен, как легко теперь можно размещать приложения на Dart на [Heroku](http://heroku.com).

Чтобы запуститься на Heroku ваше приложение должно настраиваться через переменные окружения. Каждое приложение должно как минимум брать из переменных окружения номер порта, на котором должен работать сервер. Давайте посмотрим пример:

```dart

import "dart:io";

import "package:redstone/server.dart" as app;

// Импортируем сервисы

main(List<String> args) {

  app.setupConsoleLog();

  // возьмем переменную окружения
  var port = _getConfig("PORT", "8080");

  // запустим сервер
  app.start(port: int.parse(port));
}

_getConfig(String name, [defaultValue]) {
  var value = Platform.environment[name];
  if (value == null) {
    return defaultValue;
  }
  return value;
}

```

Если ваше приложение имеет клиентский код, то было бы хорошо сделать так, чтобы путь к публичной папке можно было тоже настроить через переменные окружения.

```dart
import "dart:io";

import "package:redstone/server.dart" as app;
import "package:shelf_static/shelf_static.dart";

// Импортируем сервисы

main(List<String> args) {

  app.setupConsoleLog();

  // возьмем переменные окружения
  var port = _getConfig("PORT", "8080");
  var web = _getConfig("WEB_FOLDER", "web");

  // запустим сервер
  app.setShelfHandler(createStaticHandler(web,
                      defaultDocument: "index.html"));
  app.start(port: int.parse(port));
}

_getConfig(String name, [defaultValue]) {
  var value = Platform.environment[name];
  if (value == null) {
    return defaultValue;
  }
  return value;
}


```

Когда вы отправляете Dart-приложение на Heroku, то запускается команда `pub build` для компиляции клиентского кода, после чего запускается сервер. Для того, чтобы Heroku смог запустить ваш сервер, нужно создать `Procfile`, в котором будет написана команда для запуска. Например, если точка входа в ваш сервер находится в файле `bin/server.dart`, то содержимое `Procfile` должно выглядеть так:

```
web: ./dart-sdk/bin/dart bin/server.dart
```

Для настройки развертывания приложения вы можете использовать [Heroku Toolbelt](https://toolbelt.heroku.com/). Для того, чтобы начать его использовать, просто введите следующие команды в корневой директории вашего проекта.

Первым делом нужно сделать git-репозиторий. Если он уже создан, то можете пропустить этот шаг:

```
$ git init
$ git add .
$ git commit -am "first commit"
```

Создайте Heroku-приложение через cedar-14:

```
$ heroku create -s cedar-14
```

Укажите архив с Dart SDK. Следующия ссылка веден на Dart SDK 1.7-dev.4.6, но вы можете использовать другую версию [отсюда](https://www.dartlang.org/tools/download_archive/) (убедитесь, что выбрали 64-битную версию для Linux):

```
$ heroku config:set DART_SDK_URL=https://storage.googleapis.com/dart-archive/channels/dev/release/41090/sdk/dartsdk-linux-x64-release.zip
```

Настройте Dart buildpack:

```
$ heroku config:add BUILDPACK_URL=https://github.com/igrigorik/heroku-buildpack-dart.git
```

Укажите путь к директории с клиентским кодом:

```
$ heroku config:set WEB_FOLDER=build/web
```

Наконец, сделайте push приложения в Heroku:

```
$ git push heroku master
```

Если все пройдет нормально, то ваше приложение будет готово к использованию. Вы можете увидеть URL в выводе команды `git push`. Также можно получить доступ к приложению через панель Heroku.

Для получения более подробной ифнормации почитайте документацию к [Heroku](https://devcenter.heroku.com/articles/how-heroku-works) и [Dart buildpack](https://github.com/igrigorik/heroku-buildpack-dart). Вы также можете посмотретть рабочий пример [здесь](https://github.com/luizmineo/io_2014_contacts_demo).
