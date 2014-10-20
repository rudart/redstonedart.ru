---
layout: doc
menu_item: doc
title: API плагинов
prev: Importing-libraries
next: Shelf-Middlewares
---
Плагины Redstone могут динамически создавать новые маршруты, перехватчиков, обработчиков событий, провайдеров параметров и обработчиков ответа.

**Примечание: посмотрите [redstone_mapper](https://github.com/luizmineo/redstone_mapper) - это завершенный плагин для сериализации**

Например, если вашему приложению часто нужно конвертировать данные из json в объекты Dart, то можно делать что-то вроде этого:

```dart
@app.Route("/user", methods: const[app.POST])
printUser(@app.Body() Map json) {
  User user = new User();
  user.fromJson(json);
  ...
}
```

Вы можете сделать плагин, который выполнит эту работу за вас. Пример:

```dart
class FromJson {
  
  const FromJson();
  
}

FromJsonPlugin(app.Manager manager) {
  
  manager.addParameterProvider(FromJson, (metadata, type, handlerName, paramName, req, injector) {
    if (req.bodyType != app.JSON) {
      throw new app.RequestException(
          "FromJson плагни - $handlerName", "content-type должен быть 'application/json'");
    }
    
    ClassMirror clazz = reflectClass(type);
    InstanceMirror obj = clazz.newInstance(const Symbol(""), const []);
    obj.invoke(#fromJson, [req.body]);
    return obj.reflectee;
  });
  
}
```
Теперь, если вы установите плагин `FromJsonPlugin`, то сможете использовать аннотацию `@FromJson`:

```dart
@app.Route("/user", methods: const[app.POST])
printUser(@FromJson() User user) {
  ...
}

main() {
  app.addPlugin(FromJsonPlugin);
  app.setupConsoleLog();
  app.start();
}
```

Кроме того, вы можете сделать плагин, который сконвертирует ваши объекты Dart в json:

```dart
class ToJson {
  
  const ToJson();
  
}

ToJsonPlugin(app.Manager manager) {
  manager.addResponseProcessor(ToJson, (metadata, handlerName, value, injector) {
    if (value == null) {
      return value;
    }
    return value.toJson();
  });
}
```

```dart
@app.Route("/user/find")
@ToJson()
findUser() {
  return new Future(() {
    ...
    return user;
  });
}

main() {
  app.addPlugin(ToJsonPlugin);
  app.setupConsoleLog();
  app.start();
}
```