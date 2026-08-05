---
tags: [spring, springmvc]
---

# Handler Mapping

**Handler Mapping** — компонент Spring MVC, хранящийся в [[Servlet WebApplicationContext]]. Сопоставляет входящий HTTP-запрос с конкретным обработчиком на основе URL и другой мета-информации. Ссылку на него хранит [[DispatcherServlet]] ([[Servlet WebApplicationContext#^DispatcherServlet|через Servlet WebApplicationContext]]).

## Контракт

```java
public interface HandlerMapping {
    HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception;
}
```

`HandlerExecutionChain` = сам обработчик + цепочка [[Handle Interceptor|interceptors]].

## Результат

По результату сопоставления `Handler Mapping` возвращает [[Handle Object]] (обёртку над [[@Controller]]) и цепочку [[Handle Interceptor]], которые далее передаются в [[Handler Adapter]].

## Основные реализации

- **`RequestMappingHandlerMapping`** — работа с [[@RequestMapping]] и его специализациями (`@GetMapping`, `@PostMapping`, …). Стандарт в современных Spring-приложениях.
- **`SimpleUrlHandlerMapping`** — прямое статическое сопоставление URL → контроллер. Используется реже (для ресурсов, статических страниц).
- **`BeanNameUrlHandlerMapping`** — сопоставление URL по имени бина контроллера (устаревший).
- **`RouterFunctionMapping`** — для functional routing (Spring 5+, WebFlux-style).

## Регистрация @RequestMapping

`RequestMappingHandlerMapping` при инициализации сканирует все [[@Controller]] и собирает карту URL → метод на основе аннотаций [[@RequestMapping]]. При запросе он ищет в этой карте подходящий под паттерн и условия (HTTP-метод, заголовки, consumes/produces) обработчик.

## Пример: что видно в Map (печатается через `RequestMappingHandlerMapping`)

```
GET /users         → UserController#list()
GET /users/{id}    → UserController#show(Long)
POST /users        → UserController#create(UserCreateRequest)
PUT /users/{id}    → UserController#update(Long, UserUpdateRequest)
DELETE /users/{id} → UserController#delete(Long)
```

## Кастомный HandlerMapping

```java
@Component
public class CustomHandlerMapping implements HandlerMapping {
    @Override
    public HandlerExecutionChain getHandler(HttpServletRequest req) {
        if (req.getRequestURI().startsWith("/custom/")) {
            return new HandlerExecutionChain(customHandler, interceptors);
        }
        return null; // дать шанс другим HandlerMapping
    }
}
```

Порядок нескольких mapping'ов задаётся через `setOrder()` — первый вернул не-null, тот и выиграл.

friend:: [[Handle Object]]
friend:: [[@RequestMapping]]
friend:: [[DispatcherServlet]]
friend:: [[Handler Adapter]]
friend:: [[Servlet WebApplicationContext]]