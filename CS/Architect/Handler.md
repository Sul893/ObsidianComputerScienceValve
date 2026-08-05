---
tags: [architect]
---

# Handler

**Handler** — обобщённое название классов, выполняющих одну определённую функцию: обработку входных данных, событий или запросов. Широко используется в событийно-ориентированной архитектуре (event-driven) и в каркасах веб-фреймворков.

В отличие от контроллера (который специализируется на HTTP), `Handler` — общее понятие: любой компонент, принимающий сообщение и исполняющий логику.

## Виды обработчиков

### EventHandler

Объект, реагирующий на события предметной области.

```java
public class UserCreatedHandler implements EventHandler<UserCreated> {
    @Override
    public void on(UserCreated event) {
        log.info("Пользователь создан: {}", event.username());
    }
}
// Регистрируется в шине событий:
eventBus.subscribe(UserCreated.class, new UserCreatedHandler());
// Событие публикуется из сервиса:
eventBus.publish(new UserCreated("alice", "alice@example.com"));
```

### Command Handler

Обработчик команды в CQRS/командной модели — каждую команду (intent) обрабатывает отдельный handler.

```java
public class CreateOrderHandler implements CommandHandler<CreateOrder> {
    private final OrderService service;
    public CreateOrderHandler(OrderService s) { this.service = s; }

    @Override
    public void handle(CreateOrder cmd) {
        service.create(cmd.userId(), cmd.items());
    }
}
// Dispatcher находит подходящий handler по типу команды:
dispatcher.dispatch(new CreateOrder(user, List.of(item)));
```

### Request Handler

 Обработчик HTTP-запросов — особый случай: handler, специализированный под HTTP-протокол.

```java
public class StaticFileHandler implements HttpRequestHandler {
    @Override
    public void handleRequest(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.setContentType("application/xml");
        resp.getWriter().write("<root/>");
    }
}
```

### Callback Handler

В контексте асинхронной обработки: callback, который вызывается при завершении операции.

```java
CompletableFuture<String> f = CompletableFuture.supplyAsync(() -> "result");
f.thenAccept(result -> System.out.println("Handler получил: " + result));
```

## Связанные понятия в каркасах

В типичном веб-фреймворке вокруг `Handler` строится инфраструктура:

- **Handler Mapping** — выбирает подходящий handler под запрос
- **Handler Adapter** — исполняет handler, абстрагируя тип (через паттерн Adapter)
- **Handler Interceptor** — пред-/постобработка вокруг handler'а (через Chain of Responsibility)
- **HandlerMethod** — обёртка над конкретным методом-обработчиком (инкапсулирует `Method`, параметры, правила binding'а)

Например `eventHandler` — объект, обрабатывающий события:

```java
public class EventHandler {
    public void handleUserCreated(UserCreatedEvent event) {
        // реакция на событие
    }
}
```

![[HANDLERImg.excalidraw]]