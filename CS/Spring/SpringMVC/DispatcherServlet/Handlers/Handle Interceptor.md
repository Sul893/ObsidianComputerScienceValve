---
tags: [spring, springmvc]
---

# Handle Interceptor

**Handler Interceptor** — аналог [[Servlet filter]] в Spring MVC. Выполняет проверки и дополнительную обработку "вокруг" вызова [[Handler Adapter]], не вторгаясь в логику [[@Controller]].

В отличие от Servlet Filter, работает выше — уже внутри [[DispatcherServlet]], и имеет доступ к Spring-контексту и [[Handle Object]].

## Контракт

```java
public interface HandlerInterceptor {
    default boolean preHandle(HttpServletRequest req,
                               HttpServletResponse resp,
                               Object handler) throws Exception { return true; }

    default void postHandle(HttpServletRequest req,
                            HttpServletResponse resp,
                            Object handler,
                            ModelAndView mv) throws Exception {}

    default void afterCompletion(HttpServletRequest req,
                                 HttpServletResponse resp,
                                 Object handler,
                                 Exception ex) throws Exception {}
}
```

## Методы

`HandlerInterceptor` определяет три точки перехвата:

- **`preHandle(request, response, handler)`** — вызывается **до** исполнения контроллера. Возвращает `boolean`: если `false`, обработка прерывается и последующие перехватчики и сам контроллер не вызываются. Типовое применение: проверка аутентификации, авторизации, валидации заголовков
- **`postHandle(request, response, handler, modelAndView)`** — вызывается **после** контроллера, но **до** рендеринга [[ViewResolver]]. Здесь можно модифицировать модель, добавив атрибуты
- **`afterCompletion(request, response, handler, ex)`** — вызывается **после** рендеринга, при отправке ответа. Используется для очистки ресурсов (`ThreadLocal`), логирования; сюда попадает исключение, если оно возникло

## Порядок

`preHandle` вызывается в порядке объявления перехватчиков; `postHandle`/`afterCompletion` — в обратном порядке (LIFO).

## Пример: interceptor логирования + JWT

```java
public class JwtAuthInterceptor implements HandlerInterceptor {
    private final JwtService jwt;

    public JwtAuthInterceptor(JwtService jwt) { this.jwt = jwt; }

    @Override
    public boolean preHandle(HttpServletRequest req,
                             HttpServletResponse resp,
                             Object handler) {
        String header = req.getHeader("Authorization");
        if (header == null || !header.startsWith("Bearer ")) {
            resp.setStatus(401);
            return false; // прервать цепочку — контроллер не вызывается
        }
        String token = header.substring(7);
        UserDetails user = jwt.parse(token);
        SecurityContextHolder.getContext().setAuthentication(
            new UsernamePasswordAuthenticationToken(user, null, user.getAuthorities()));
        return true; // продолжить
    }

    @Override
    public void afterCompletion(HttpServletRequest req,
                                 HttpServletResponse resp,
                                 Object handler, Exception ex) {
        SecurityContextHolder.clearContext(); // очистка ThreadLocal
    }
}
```

## Регистрация

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    private final JwtAuthInterceptor jwt;

    public WebConfig(JwtAuthInterceptor jwt) { this.jwt = jwt; }

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(jwt)
                .addPathPatterns("/api/**")
                .excludePathPatterns("/api/login", "/api/register");
    }
}
```

## Сравнение с Servlet Filter

| Критерий | Servlet Filter | Handle Interceptor |
|----------|----------------|---------------------|
| Уровень | контейнер сервлетов | Spring MVC (DispatcherServlet) |
| Доступ к Spring-контексту | нет (без DelegatingFilterProxy) | есть |
| Видит Handle Object | нет | да |
| Может прервать запрос | да | да |
| Может изменять ModelAndView | нет | да (postHandle) |
| Производительность | ниже (до Spring) | выше (не для стат. ресурсов) |

## Async interceptor

Для `DeferredResult`/`Callable`/`Mono` есть `AsyncHandlerInterceptor` с методом `afterConcurrentHandlingStarted`.

friend:: [[Handler Adapter]]
friend:: [[Handler Mapping]]
friend:: [[Servlet filter]]
friend:: [[DispatcherServlet]]
friend:: [[@Controller]]
friend:: [[Handle Object]]
friend:: [[ViewResolver]]