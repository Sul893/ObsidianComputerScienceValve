---
tags: [spring, springmvc]
---

# Root WebApplicationContext

**Root WebApplicationContext** — реализация [[ApplicationContext]] для корневого уровня веб-приложения. Содержит общие (не веб-специфичные) бины: сервисы, репозитории, инфраструктуру, конфигурацию безопасности.

## Содержимое

- [[RootConfigurer]] — главный конфигурационный класс с [[@ComponentScan]] и [[@Configuration]]-аннотациями
- `@Service`, `@Repository`-бины
- Инфраструктурные бины, такие как [[ViewResolver]]
- При наличии — конфигурация [[SecurityConfig]] (Spring Security)
- `DataSource`, `TransactionManager`, cache, converters, ...

## Иерархия

Является **родительским** по отношению к [[Servlet WebApplicationContext]]. Дочерний контекст (создаваемый [[DispatcherServlet]]) может получать отсюда бины, но не наоборот: родитель не видит дочерних бинов (контроллеров, handler mappings).

```
Root WebApplicationContext (сервисы, репозитории, security)
   ↑ parent
   │
   ├── Servlet WebApplicationContext (DispatcherServlet 1, REST)
   │
   └── Servlet WebApplicationContext (DispatcherServlet 2, admin)
```

Это даёт изоляцию: один root — общая бизнес-логика; несколько Servlet-контекстов — для разных DispatcherServlet (например, REST API + admin UI) с разными веб-конфигами.

## Создание

Создаётся [[ContextLoaderListener]] при запуске [[Apache Tomcat]] ([[Web Container]]). Инициализация происходит раньше, чем создание [[DispatcherServlet]].

```java
// WebApplicationInitializer
public void onStartup(ServletContext container) {
    AnnotationConfigWebApplicationContext root = new AnnotationConfigWebApplicationContext();
    root.register(RootConfigurer.class, SecurityConfig.class);
    root.setServletContext(container);
    root.refresh();

    container.addListener(new ContextLoaderListener(root));

    // DispatcherServlet инициализируется ниже и возьмёт root как parent
    DispatcherServlet ds = new DispatcherServlet();
    ServletRegistration.Dynamic app = container.addServlet("app", ds);
    app.setLoadOnStartup(1);
    app.addMapping("/");
}
```

## Доступ к Root из произвольного кода

```java
ServletContext sc = ...;
WebApplicationContext root = (WebApplicationContext)
    sc.getAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE);
```

friend:: [[ContextLoaderListener]]
friend:: [[RootConfigurer]]
friend:: [[Servlet WebApplicationContext]]
friend:: [[DispatcherServlet]]
friend:: [[Apache Tomcat]]
friend:: [[Web Container]]
friend:: [[@ComponentScan]]
friend:: [[@Configuration]]
friend:: [[ViewResolver]]
friend:: [[SecurityConfig]]
parent:: [[ApplicationContext]]