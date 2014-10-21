---
layout: doc
menu_item: doc
title: redstone_web_socket
prev: redstone_mapper_pg
---
[redstone_web_socket](http://pub.dartlang.org/packages/redstone_web_socket) - это веб-сокет плагин для Redstone.dart. Для создания обработчиков он использует пакет [shelf_web_socket](http://pub.dartlang.org/packages/shelf_web_socket).

### Использование @WebSocketHandler с функциями

Если у функции есть аннотация `@WebSocketHandler`, то она будет вызвана с объектом [CompatibleWebSocket](https://api.dartlang.org/apidocs/channels/be/dartdoc-viewer/http_parser/http_parser.CompatibleWebSocket) для каждого нового соединения:

```dart
@WebSocketHandler("/ws")
onConnection(websocket) {
  websocket.listen((message) {
    websocket.add("echo $message");
  });
}
```

### Использование @WebSocketHandler с классами

Если у класса есть аннотация `@WebSocketHandler`, то плагин установит слушателей событий на каждом методе с аннотациями `@OnOpen`, `@OnMessage`, `@OnError` и `@OnClose`:

```dart

@WebSocketHandler("/ws")
class ServerEndPoint {

  @OnOpen()
  void onOpen(WebSocketSession session) {
    print("connection established");
  }

  @OnMessage()
  void onMessage(String message, WebSocketSession session) {
    print("message received: $message");
    session.connection.add("echo $message");
  }

  @OnError()
  void onError(error, WebSocketSession session) {
    print("error: $error");
  }

  @OnClose()
  void onClose(WebSocketSession session) {
    print("connection closed");
  }

}

```

Подобно [группам](/doc/Groups.html), класс будет инициализирован только один раз и он может подключить зависимости в конструкторе ([внедрение зависимостей](/doc/Dependency-Injection.html)).

### Установка обработчиков

Для установки веб-сокет обработчиков, вам нужно просто импортировать `redstone_web_socket.dart` и вызвать `getWebSocketPlugin()`:

```dart
import 'package:redstone/server.dart' as app;
import 'package:redstone_web_socket/redstone_web_socket.dart';

void main() {
  app.setupConsoleLog();
  
  // устанавливаем обработчиков
  app.addPlugin(getWebSocketPlugin());
  
  app.start();
}
```

### Тестирование

Этот пакет также предоставляет простого клиента, который может быть использован для тестирования:

```dart
import 'package:redstone/server.dart' as app;
import 'package:redstone_web_socket/redstone_web_socket.dart';
import 'package:unittest/unittest.dart';

main() {
  
  test("Test echo service", () {
  
    var completer = new Completer();
    var socket = new MockWebSocket();
    
    socket.listen((message) {
      
      expect(message, equals("echo message"));
      
      completer.complete();
    });
    
    openMockConnection("/ws", socket);
    
    socket.add("message");
    
    return completer.future;
  
  });

}
```