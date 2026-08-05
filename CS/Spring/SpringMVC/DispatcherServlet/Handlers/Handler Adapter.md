---
tags: [spring, springmvc]
---

# Handler Adapter

**Handler Adapter** — адаптер в Spring MVC, принимающий [[Handle Object]] у [[DispatcherServlet]] (после выбора обработчика [[Handler Mapping]]) и исполняющий "реальный код" контроллера.

## Контракт

```java
public interface HandlerAdapter {
    boolean supports(Object handler);                           // подходит ли для handler
    ModelAndView handle(HttpServletRequest req,
                       HttpServletResponse resp,
                       Object handler) throws Exception;      // вызывает handler
    long getLastModified(HttpServletRequest req, Object handler);
}
```

## Зачем нужен

[[DispatcherServlet]] не знает, какого типа экземпляр контроллера обрабатывает: это может быть аннотированный [[@Controller]], реализующий интерфейс `Controller`, или `HttpRequestHandler`. `Handler Adapter` абстрагирует разницу — предоставляет единый интерфейс для вызова любого обработчика через паттерн Adapter.

Адаптер принимает Handle Object и мета-информацию, затем вызывает `invoke()` на нём, тем самым исполняя логику контроллера. Возвращённое контроллером значение (имя представления, `ModelAndView`, или return value для REST) передаётся обратно в `DispatcherServlet`.

## Порядок выполнения

Перед вызовом и после отрабатывают [[Handle Interceptor|Handle Interceptors]] (`preHandle`, `postHandle`), которые могут прервать цепочку или обрабатывать результат.

```
preHandle()      → если false — прервать
[Handler Adapter] invokes @RequestMapping method
postHandle()     → модифицировать ModelAndView
после рендеринга:
afterCompletion()  → очистка/логирование
```

## Основные реализации

- **`RequestMappingHandlerAdapter`** — для аннотированных контроллеров ([[@RequestMapping]]), стандарт
- **`SimpleControllerHandlerAdapter`** — для классов `implements Controller` (устаревший интерфейс from Spring 2)
- **`HttpRequestHandlerAdapter`** — для `HttpRequestHandler` (`/resources/**` etc)
- **`HandlerFunctionAdapter`** — для функциональных роутеров (`RouterFunction`)

## Что делает RequestMappingHandlerAdapter внутри

```
RequestMappingHandlerAdapter.handle(req, resp, handlerMethod):
   1. Определяет и подготавливает аргументы метода
        (через HandlerMethodArgumentResolver: @PathVariable, @RequestParam, @RequestBody, ...)
   2. Инвокает java.lang.reflect.Method.invoke(bean, args)
   3. Обрабатывает return value через HandlerMethodReturnValueHandler
        (String viewName, @ResponseBody → Jackson,
         ModelAndView, ResponseEntity, Mono, ...)
   4. Возвращает ModelAndView (null для REST/void)
```

## Пример: кастомный HandlerAdapter (для нестандартных controller)

```java
public class MyControllerAdapter implements HandlerAdapter {
    @Override
    public boolean supports(Object handler) {
        return handler instanceof MyController;
    }
    @Override
    public ModelAndView handle(HttpServletRequest req,
                                HttpServletResponse resp,
                                Object handler) throws Exception {
        ((MyController) handler).execute(req, resp);
        return null; // сам пишет в response
    }
    @Override
    public long getLastModified(HttpServletRequest req, Object handler) {
        return -1;
    }
}
```

friend:: [[Handler Mapping]]
friend:: [[Handle Object]]
friend:: [[Handle Interceptor]]
friend:: [[DispatcherServlet]]
friend:: [[@Controller]]
friend:: [[@RequestMapping]]