---
tags: [spring, springmvc, web]
---

# ContextLoaderListener

**`ContextLoaderListener`** — `ServletContextListener`, который [[Apache Tomcat]] ([[ServletContainer]]) запускает при инициализации веб-приложения (до того, как создаются сервлеты).

## Что делает

1. Создаёт [[Root WebApplicationContext]] — реализацию [[ApplicationContext]].
2. Загружает из [[RootConfigurer]] конфигурацию (через [[@Configuration]] и [[@ComponentScan]]).
3. Помещает созданный контекст в [[ServletContext]] как атрибут, делая его доступным для всех сервлетов приложения.

## Контракт ServletContextListener

```java
public interface ServletContextListener extends EventListener {
    void contextInitialized(ServletContextEvent sce); // при старте app
    void contextDestroyed(ServletContextEvent sce);    // при остановке
}
```

ContextLoaderListener при старте:

```java
public void contextInitialized(ServletContextEvent event) {
    ServletContext sc = event.getServletContext();
    WebApplicationContext root =
        ContextLoader.createWebApplicationContext(sc);
    sc.setAttribute(
        WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE,
        root);
    // → из любого сервлета: sc.getAttribute(ROOT_WEB_APPLICATION..._CONTEXT...)
}
```

## Взаимодействие с DispatcherServlet

`ContextLoaderListener` срабатывает **до** создания [[DispatcherServlet]]. Когда DispatcherServlet инициализируется, он создаёт свой [[Servlet WebApplicationContext]] и устанавливает в качестве родителя уже готовый Root-контекст — это даёт веб-компонентам доступ к сервисам и репозиториям.

## Регистрация

### Через web.xml (classic)

```xml
<listener>
    <listener-class>
        org.springframework.web.context.ContextLoaderListener
    </listener-class>
</listener>

<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:com/example/RootConfigurer.class</param-value>
</context-param>
```

### Программно (Spring Boot / Servlet 3+)

```java
public class WebAppInitializer implements WebApplicationInitializer {
    @Override
    public void onStartup(ServletContext ctx) {
        AnnotationConfigWebApplicationContext root =
            new AnnotationConfigWebApplicationContext();
        root.register(RootConfigurer.class, SecurityConfig.class);
        root.refresh();

        ctx.addListener(new ContextLoaderListener(root));
        // DispatcherServlet создаётся ниже, см. [DispatcherServlet]
    }
}
```

## В Spring Boot не используется явно

Spring Boot собирает приложение через `SpringApplication.run()` + `SpringBootServletInitializer` — Root и Servlet контексты создаются автоматически, `ContextLoaderListener` прячется в недрах `SpringServletContainerInitializer`.

friend:: [[Apache Tomcat]]
friend:: [[Root WebApplicationContext]]
friend:: [[ServletContext]]
friend:: [[ApplicationContext]]
friend:: [[ServletContainer]]
friend:: [[RootConfigurer]]
friend:: [[@Configuration]]
friend:: [[@ComponentScan]]
friend:: [[DispatcherServlet]]
friend:: [[Servlet WebApplicationContext]]