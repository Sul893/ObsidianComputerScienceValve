---
tags: [spring, springmvc]
---

# Handle Object

**Handle Object** (также называется *handler*, *handler method*) — обёртка над [[@Controller]], создаваемая Spring MVC при обработке запроса. Инкапсулирует сам объект контроллера и мета-информацию о конкретном методе [[@RequestMapping]], который должен быть вызван для обработки HTTP-запроса.

В Spring аналогом служит класс `HandlerMethod` (из Spring 3.1+).

## Что хранит

```java
public class HandlerMethod {
    private final Object bean;            // экземпляр @Controller
    private final Method method;          // java.lang.reflect.Method
    private final MethodParameter[] parameters; // параметры (для binding)
    // + аннотации, bean factory, ...
}
```

## Откуда берётся

`Handle Object` формируется [[Handler Mapping]] на основе URL-паттерна и условий HTTP-запроса. По найденному `@RequestMapping`-методу Spring определяет:
- Объект контроллера
- Метод (`Method` reference)
- Параметры метода и правила их связывания (из `@RequestParam`, `@PathVariable`, `@RequestBody` и т.д.)
- Цепочку [[Handle Interceptor]]

После формирования Handle Object передаётся в [[Handler Adapter]], который выполняет метод `invoke()` для вызова реального кода контроллера.

## Место в жизненном цикле

```
HTTP-запрос
   ↓
[DispatcherServlet]
   ↓ getHandler(request)
[Handler Mapping] → возвращает HandleExecutionChain
   │                 ├─ Handle Object (HandlerMethod)
   │                 └─ List<HandlerInterceptor>
   ↓
[Handler Adapter] → invoke() → Controller.return "users/detail"
   ↓
[ViewResolver] → /WEB-INF/views/users/detail.jsp
   ↓
[HTML ответ]
```

## Пример: доступ к Handle Object в interceptor

```java
public class LoggingInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest req,
                             HttpServletResponse resp,
                             Object handler) {
        if (handler instanceof HandlerMethod hm) {
            System.out.printf("Calling %s.%s()%n",
                hm.getBeanType().getSimpleName(), hm.getMethod().getName());
            // Calling UserController.show()
        }
        return true;
    }
}
```

friend:: [[@Controller]]
friend:: [[Handler Mapping]]
friend:: [[Handler Adapter]]
friend:: [[DispatcherServlet]]
friend:: [[@RequestMapping]]
friend:: [[Handle Interceptor]]