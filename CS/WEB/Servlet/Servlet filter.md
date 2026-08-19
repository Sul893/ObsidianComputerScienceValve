---
tags: [web, servlet]
---

# Servlet Filter

**Servlet Filter** — фильтр, выполняющий пред- и пост-обработку HTTP-запросов перед тем, как они попадут в [[Servlet]] (и после обработки сервлетом). Работает на уровне [[Apache Tomcat]] ([[Web Container]]) — ниже уровня сервлетов.

Интерфейс: `jakarta.servlet.Filter`, жизненный цикл `init()` → `doFilter()` → `destroy()`.

## Назначение

Цепочка фильтров (Filter Chain) позволяет вынести сквозную (cross-cutting) функциональность из сервлетов:

- **Аутентификация и авторизация** — проверка прав до попадания в сервлет
- **Логирование и аудит** — запись входящих запросов и исходящих ответов
- **Сжатие** — gzip перед отправкой клиенту
- **Модификация заголовков** — добавление `Content-Type`, `X-Frame-Options` и др.
- **Валидация и преобразование** — нормализация запроса

Фильтры вызываются контейнером в порядке объявления. Каждый фильтр может **прервать** цепочку, не передав запрос дальше (`chain.doFilter()` не вызывается).

## Контракт

```java
public interface Filter {
    default void init(FilterConfig filterConfig) throws ServletException {}
    void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain)
        throws IOException, ServletException;
    default void destroy() {}
}
```

## Пример: фильтр логирования

```java
public class LoggingFilter implements Filter {
    @Override
    public void doFilter(ServletRequest req, ServletResponse resp,
                         FilterChain chain) throws IOException, ServletException {
        HttpServletRequest  request  = (HttpServletRequest) req;
        HttpServletResponse response = (HttpServletResponse) resp;
        long start = System.nanoTime();

        try {
            chain.doFilter(request, response); // передать дальше
        } finally {
            long ms = (System.nanoTime() - start) / 1_000_000;
            System.out.printf("%s %s -> %d (%d ms)%n",
                request.getMethod(), request.getRequestURI(),
                response.getStatus(), ms);
        }
    }
}
```

## Регистрация

### Через web.xml

```xml
<filter>
    <filter-name>logging</filter-name>
    <filter-class>com.example.LoggingFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>logging</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

### Аннотацией

```java
@WebFilter(urlPatterns = "/*")
public class LoggingFilter implements Filter { ... }
```

### Программно (Servlet 3+)

```java
public class WebInit implements WebApplicationInitializer {
    public void onStartup(ServletContext container) {
        FilterRegistration.Dynamic f = container.addFilter("logging", new LoggingFilter());
        f.addMappingForUrlPatterns(null, false, "/*");
    }
}
```

## Цепочка фильтров

```java
// FilterChain внутри (упрощённо):
chain.doFilter(request, response);
// Tomcat проходит по Filters[0..n] → Servlet → обратно изнутри наружу:
// [Filter0.doFilter-before] → [Filter1.doFilter-before] → Servlet
//                           → [Filter1.doFilter-after]
//                           → [Filter0.doFilter-after]
```

## Сравнение с перехватчиком из MVC-фреймворков

MVC-фреймворки ( поверх контейнера сервлетов) часто предлагают свою абстракцию — *interceptor* — с аналогичной задачей:

| Критерий | Servlet Filter | MVC Interceptor |
|----------|----------------|-----------------|
| Уровень | контейнер сервлетов | уровень MVC-фреймворка |
| Видит handler-объект | нет | да |
| Может изменить ModelAndView | нет | да (postHandle) |
| Ранний доступ к запросу | да | нет |
| Время вызова | до сервлета | после выбора handler'а |

friend:: [[Apache Tomcat]]
friend:: [[Web Container]]
friend:: [[Servlet]]