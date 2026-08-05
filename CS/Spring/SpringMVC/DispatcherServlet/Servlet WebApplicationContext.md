---
tags: [spring, springmvc]
---

# Servlet WebApplicationContext

**Servlet WebApplicationContext** — дочерний контекст в иерархии Spring, создаваемый каждым [[DispatcherServlet]]. Управляет веб-компонентами приложения.

## Содержимое

- [[@Controller]] — контроллеры веб-слоя
- [[Handler Mapping]] — маппинг URL на контроллеры
- [[Handler Adapter]] — адаптеры для вызова контроллеров
- [[ViewResolver]] (если не вынесен в root) — резолверы представлений
- [[Handle Interceptor]]'ы, `Validator`, `MultipartResolver`, `LocaleResolver`

## Иерархия

[[Root WebApplicationContext]] является для него **родителем**. Дочерний контекст наследует доступ к общим бинам (сервисам, репозиториям), но родительский контекст не видит содержимое дочернего.

```
Root WebApplicationContext (сервисы, репозитории, security)
   ↑ parent
   │
   ├── Servlet WebApplicationContext (DispatcherServlet 1, REST) ←?;
   │
   └── Servlet WebApplicationContext (DispatcherServlet 2, admin)
```

Это даёт изоляцию: один root — общая бизнес-логика; несколько Servlet-контекстов — для разных DispatcherServlet (например, REST API + admin UI) с разными веб-конфигами.

## Создание

Создаётся [[DispatcherServlet]] при его инициализации. ^DispatcherServlet

```java
// DispatcherServlet.initWebApplicationContext() (internal):
WebApplicationContext servletCtx = ...;
// берём Root из ServletContext атрибута
WebApplicationContext root = (WebApplicationContext)
    getServletContext().getAttribute(ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE);
servletCtx.setParent(root);   // вот тут иерархия
```

## Программная установка (WebApplicationInitializer)

```java
public class WebInitializer implements WebApplicationInitializer {
    @Override
    public void onStartup(ServletContext container) {
        // 1. Root context
        AnnotationConfigWebApplicationContext root = new AnnotationConfigWebApplicationContext();
        root.register(RootConfigurer.class);
        container.addListener(new ContextLoaderListener(root));

        // 2. Servlet context (дочерний)
        AnnotationConfigWebApplicationContext web = new AnnotationConfigWebApplicationContext();
        web.register(WebConfigurer.class);
        DispatcherServlet dispatcher = new DispatcherServlet(web);
        ServletRegistration.Dynamic reg =
            container.addServlet("dispatcher", dispatcher);
        reg.setLoadOnStartup(1);
        reg.addMapping("/");
    }
}

@Configuration
@ComponentScan("com.example.web") // только контроллеры и веб-бины
public class WebConfigurer { ... }
```

## Доступ к контексту

```java
// Из контроллера:
@Autowired
private ApplicationContext servletCtx; // это Servlet WebApplicationContext

// Через ServletContext — Root
WebApplicationContext root = (WebApplicationContext)
    request.getServletContext()
           .getAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE);
```

friend:: [[DispatcherServlet]]
friend:: [[@Controller]]
friend:: [[Handler Mapping]]
friend:: [[Root WebApplicationContext]]
friend:: [[Handler Adapter]]
friend:: [[ViewResolver]]
friend:: [[Handle Interceptor]]
parent:: [[ApplicationContext]]