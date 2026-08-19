---
tags: [web, servlet]
---

# ServletContext

**ServletContext** — общее контекстное окружение для всех [[Servlet]] одного веб-приложения, предоставляемое [[Apache Tomcat]] ([[Web Container]]).

Один `ServletContext` = одно веб-приложение (WAR). Все сервлеты приложения делят один контекст.

## Что хранит

- Глобальные **параметры инициализации** приложения (задаются в `web.xml` через `<context-param>`)
- **Атрибуты**, доступные всем сервлетам (разделяемое состояние в памяти)
- Информацию о сервере (версия, MIME-типы)
- Ресурсы приложения (пути к файлам, URL ресурсов внутри WAR; методы `getResource()`, `getResourceAsStream()`)
- Возможность регистрировать сервлеты, фильтры и слушатели программно (`addServlet()`, `addFilter()`)

## Пример: параметры через web.xml

```xml
<context-param>
    <param-name>dbUrl</param-name>
    <param-value>jdbc:postgresql://prod-db:5432/myapp</param-value>
</context-param>
```

```java
// Доступ из любого сервлета
String db = getServletContext().getInitParameter("dbUrl");

// Сохранение разделяемого объекта
getServletContext().setAttribute("cache", new Cache());
Object c = getServletContext().getAttribute("cache");
```

## Слушатели на контекст

Через `ServletContextListener` контейнер оповещает приложение о старте/остановке:

```java
public class AppBootstrapListener implements ServletContextListener {
    @Override
    public void contextInitialized(ServletContextEvent e) {
        ServletContext sc = e.getServletContext();
        sc.setInitParameter("startedAt", Instant.now().toString());
        // здесь приложение может инициализировать свои ресурсы
    }

    @Override
    public void contextDestroyed(ServletContextEvent e) {
        // освобождаем ресурсы при остановке
    }
}
```

Регистрация:

```xml
<listener>
    <listener-class>com.example.AppBootstrapListener</listener-class>
</listener>
```

## Программная регистрация сервлетов/фильтров в контексте

```java
@Override
public void contextInitialized(ServletContextEvent e) {
    ServletContext sc = e.getServletContext();
    ServletRegistration.Dynamic s = sc.addServlet("hello", HelloServlet.class);
    s.addMapping("/hello");
    s.setLoadOnStartup(1);

    FilterRegistration.Dynamic f = sc.addFilter("log", LoggingFilter.class);
    f.addMappingForUrlPatterns(null, false, "/*");
}
```

friend:: [[Apache Tomcat]]
friend:: [[Servlet]]
friend:: [[Web Container]]