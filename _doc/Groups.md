---
layout: doc
menu_item: doc
title: Группы
prev: Error-handlers
next: Dependency-Injection
---

Аннотация `@Group` используется для определения группы маршрутов, перехватчиков и обработчиков ошибок:

```dart
@app.Group("/user")
class UserService {
  
  @app.Route("/find")
  findUser(@app.QueryParam("n") String name,
           @app.QueryParam("c") String city) {
    ...
  }

  @app.Route("/add", methods: const [app.POST])
  addUser(@app.Body(app.JSON) Map json) {
    ...
  }
}
```

Префикс, указанный в аннотации `@Group` будет подставлен в начало каждого маршрута и перехватчика внутри группы. Если вам нужно связать функцию с URL, который соответствует URL группы, то используйте аннотацию `@DefaultRoute`.

В примере ниже мы определяем группу с URL `/user`. При GET-запросе на этот путь, мы выполним функцию `getUser()`; при POST-запросе, будет выполнена функция `postUser()`. Если к нам придет запрос вида `/user/:id`, то будет выполнена функция `getUserById(String id)`.

```dart
@app.Group("/user")
class UserService {
  
  @app.DefaultRoute()
  getUser() {
    ...
  }

  @app.DefaultRoute(methods: const[app.POST])
  postUser(@app.Body(app.JSON) Map user) {
    ...
  }

  @app.Route("/:id")
  getUserById(String id) {
   ...
  }
}

```

