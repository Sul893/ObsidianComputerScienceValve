---
tags: [spring, springmvc, web]
---

# DispatcherServlet

**`DispatcherServlet`** — центральный [[Servlet]] в Spring MVC, единая точка входа ("Front Controller"). [[Apache Tomcat]] ([[ServletContainer]]) создаёт его при инициализации приложения и регистрирует как обычный сервлет.

## Роль

Все HTTP-запросы к веб-приложению попадают сначала в `DispatcherServlet`. Он играет роль диспетчера: не содержит бизнес-логики, а координирует работу остальных компонентов Spring MVC.

## Инициализация

При инициализации `DispatcherServlet` создаёт [[Servlet WebApplicationContext]] — дочерний контекст, родителем которого выступает [[Root WebApplicationContext]]. В дочерний контекст кладутся веб-компоненты.

## Жизненный цикл запроса

Запрос проходит следующую цепочку:

1. [[Apache Tomcat]] принимает запрос через [[Socket]] и передаёт его в `DispatcherServlet`
2. `DispatcherServlet` обращается к [[Handler Mapping]] — находит [[Handle Object]] по URL из [[@RequestMapping]]
3. Запрос и Handle Object передаются в [[Handler Adapter]], который вызывает метод [[@Controller]] (с учётом [[Handle Interceptor]])
4. Контроллер возвращает логическое имя представления
5. [[ViewResolver]] определяет, какую [[Web page]] отобразить
6. Результат рендерится и отправляется обратно через сокет

```
HTTP-запрос
   ↓
[Apache Tomcat] → [DispatcherServlet.service()]
   │
   ├── [Handler Mapping]      → URL → Handle Object (метод @RequestMapping)
   ├── [Handle Interceptor.preHandle]
   ├── [Handler Adapter]     → вызывает метод @Controller
   ├── [Handle Interceptor.postHandle]
   ├── [ViewResolver]        → "users/list" → JSP/Thymeleaf page (или @ResponseBody для REST)
   └── [Handle Interceptor.afterCompletion]
   ↓
HTTP-ответ → Tomcat → Socket → клиент
```

## Пример: регистрация DispatcherServlet

### web.xml (classic)

```xml
<servlet>
    <servlet-name>dispatcher</servlet-name>
    <servlet-class>
        org.springframework.web.servlet.DispatcherServlet
    </servlet-class>
    <init-param>
        <param-name>contextClass</param-name>
        <param-value>
            org.springframework.web.context.support.AnnotationConfigWebApplicationContext
        </param-value>
    </init-param>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>com.example.WebConfigurer</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>dispatcher</servlet-name>
    <url-pattern>/</url-pattern>
    <!-- лишь не находятся ресурсы (/css/**etc...), которые перенаправляем -->
</servlet-mapping>
```

### Программно (Spring Boot / Servlet 3+)

```java
public class WebAppInitializer implements WebApplicationInitializer {
    @Override
    public void onStartup(ServletContext container) {
        AnnotationConfigWebApplicationContext web = new AnnotationConfigWebApplicationContext();
        web.register(WebConfigurer.class);

        DispatcherServlet servlet = new DispatcherServlet(web);
        ServletRegistration.Dynamic reg = container.addServlet("dispatcher", servlet);
        reg.setLoadOnStartup(1);
        reg.addMapping("/");
    }
}
```

### Spring Boot (всё автоматически)

```java
@SpringBootApplication
public class App {
    public static void main(String[] args) { SpringApplication.run(App.class, args); }
}
// Spring Boot регистрирует DispatcherServlet под "dispatcherServlet"
// с URL "/" по умолчанию
```

## Спец-возможности DispatcherServlet

- **Exception resolution**: любой `Exception`, брошенный из контроллера, обрабатывается `HandlerExceptionResolver`. Часто настраивается через `@ControllerAdvice` и `@ExceptionHandler`.
- **Multipart**: `MultipartResolver` парсит `multipart/form-data` (`MultipartFile`).
- **Locale**: `LocaleResolver` определяет язык пользователя.
- **Theme**: `ThemeResolver` (deprecated с Spring 6).
- **Async**: `AsyncContext` для `DeferredResult`/`Callable` (WebFlux-like).

friend:: [[Handler Mapping]]
friend:: [[Handler Adapter]]
friend:: [[ViewResolver]]
friend:: [[Servlet WebApplicationContext]]
friend:: [[Root WebApplicationContext]]
friend:: [[Apache Tomcat]]
friend:: [[ServletContainer]]
friend:: [[Socket]]
friend:: [[@RequestMapping]]
friend:: [[Handle Object]]
friend:: [[@Controller]]
friend:: [[Handle Interceptor]]
friend:: [[Web page]]
parent:: [[Servlet]]