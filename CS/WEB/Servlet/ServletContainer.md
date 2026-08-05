---
tags: [web, servlet]
---

# ServletContainer

**ServletContainer (контейнер сервлетов)** — среда выполнения для [[Servlet]]. Управляет их жизненным циклом (`init`, `service`, `destroy`), обработкой HTTP-запросов/ответов и предоставляет инфраструктурные сервисы: [[ServletContext]], пул потоков, безопасность, кэширование.

## Основные обязанности

- Регистрация и загрузка сервлетов при старте приложения (через `web.xml`, `@WebServlet` или `WebApplicationInitializer`)
- Маршрутизация входящих запросов к нужному сервлету по URL-паттерну
- Управление потоками (thread-per-request: один поток на запрос из пула)
- Поддержка фильтров ([[Servlet filter]]) и слушателей (`ServletContextListener`, `HttpSessionListener`)
- Предоставление [[ServletContext]] — общего окружения
- Управление сессиями (JSESSIONID cookie, `HttpSession`)
- Поддержка JSP, HTTPS, асинхронных запросов (`AsyncContext`)

## Внутренняя структура (на примере Catalina/Tomcat)

```
HTTP-запрос → Connector (протокол HTTP/NIO2, привязка к [Socket])
           → Engine (Catalina) → Host → Context (webapp)
           → FilterChain (фильтры) → Wrapper (Servlet instance)
           → Servlet.service(req, res)
           → Response → Connector → Socket
```

## Реализации

- [[Apache Tomcat]] — наиболее популярный
- **Jetty** — лёгкий, встраиваемый (часто используется в проектах Eclipse)
- **Undertow** (Red Hat / WildFly) — высокопроизводительный
- **Resin**, **GlassFish/Payara** (полные Jakarta EE сервера)

Встраиваемая конфигурация через SPI/Maven исключениями позволяет заменить контейнер:

```xml
<dependency>
    <groupId>jakarta.servlet</groupId>
    <artifactId>jakarta.servlet-api</artifactId>
    <scope>provided</scope> <!-- контейнер provides implementation -->
</dependency>
```

friend:: [[Servlet]]
friend:: [[Apache Tomcat]]
friend:: [[ServletContext]]
friend:: [[Socket]]
friend:: [[Servlet filter]]