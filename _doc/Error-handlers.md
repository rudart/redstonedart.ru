---
layout: doc
menu_item: doc
title: Слушатели ошибок
prev: Interceptors
next: Groups
---
Для определения слушателя ошибок используйте аннотацию `@ErrorHandler`:

```dart
@app.ErrorHandler(HttpStatus.NOT_FOUND)
handleNotFoundError() => app.redirect("/error/not_found.html");
```

Также вы можете определить обработчик ошибок для конкретного шаблона URL

```dart
@app.ErrorHandler(HttpStatus.NOT_FOUND, urlPattern: r'/public/.+')
handleNotFoundError() => app.redirect("/error/not_found.html");
```

Если вы определяете слушателя ошибок внутри группы, то обработчик будет работать только для этой группы.

```dart
@app.Group('/user')
class User {

  @app.ErrorHandler(500)
  onInternalServerError() {
    if (app.chain.error is UserException) {
      ...
    } 
  }

  @app.Route('/find')
  find() {
    ...
  }
}
```

Когда произойдет ошибка, то Redstone.dart вызовет наиболее подходящий обработчик ошибки.

