---
tags: [web]
---

# Servlet

**Сервлет** — Java-класс, обрабатывающий HTTP-запросы и генерирующий HTTP-ответы. Базовый компонент Jakarta EE (бывш. Java EE) для создания веб-приложений.

Все сервлеты реализуют интерфейс `jakarta.servlet.Servlet` (раньше `javax.servlet.Servlet`). Для HTTP-сервлетов обычно наследуются от `HttpServlet`.

## Жизненный цикл

Управляется контейнером сервлетов [[Apache Tomcat]] ([[ServletContainer]]). Жизненный цикл состоит из трёх фаз:

1. **`init()`** — инициализация. Вызывается контейнером один раз при загрузке сервлета. Здесь выполняется настройка ресурсов.
2. **`service()`** — обработка запросов. Вызывается для каждого HTTP-запроса. Метод разбирает HTTP-метод (GET, POST, …) и делегирует в `doGet()`, `doPost()` и т.д.
3. **`destroy()`** — завершение. Вызывается контейнером один раз перед выгрузкой сервлета. Здесь освобождаются ресурсы.

## Как это работает

Контейнер ([[Apache Tomcat]]) получает запрос через [[Socket]], формирует из него `HttpServletRequest` и `HttpServletResponse`, передаёт их сервлету в метод `service()`. Сервлет обрабатывает запрос, заполняет объект ответа, и контейнер отправляет ответ обратно через сокет.

## Пример сервлета

### web.xml — регистрация (классический подход)

```xml
<servlet>
    <servlet-name>hello</servlet-name>
    <servlet-class>com.example.HelloServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>hello</servlet-name>
    <url-pattern>/hello</url-pattern>
</servlet-mapping>
```

### HelloServlet.java

```java
public class HelloServlet extends HttpServlet {

    @Override
    public void init() throws ServletException {
        getServletContext().log("HelloServlet init");
    }

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {
        String name = req.getParameter("name");
        resp.setContentType("text/html;charset=UTF-8");
        try (PrintWriter w = resp.getWriter()) {
            w.write("<h1>Hello, " + (name != null ? name : "world") + "!</h1>");
        }
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {
        doGet(req, resp); // для demo — делегируем
    }

    @Override
    public void destroy() {
        getServletContext().log("HelloServlet destroy");
    }
}
```

### Аннотация `@WebServlet` (Servlet 3+, не требует web.xml)

```java
@WebServlet("/hello")
public class HelloServlet extends HttpServlet { ... }
```

### Программная регистрация (Servlet 3+ через `WebApplicationInitializer`)

```java
public class MyInitializer implements WebApplicationInitializer {
    public void onStartup(ServletContext ctx) {
        ServletRegistration.Dynamic reg =
            ctx.addServlet("hello", new HelloServlet());
        reg.addMapping("/hello");
        reg.setLoadOnStartup(1);
    }
}
```

## Совместное использование

Общие данные между сервлетами приложения хранятся в [[ServletContext]].

Сквозная обработка запросов (логирование, аутентификация) выносится в [[Servlet filter]].

friend:: [[Apache Tomcat]]
friend:: [[ServletContext]]
friend:: [[Socket]]
friend:: [[Servlet filter]]
friend:: [[ServletContainer]]