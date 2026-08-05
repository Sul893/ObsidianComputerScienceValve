---
tags: [spring, springmvc, web]
---

# ViewResolver

**ViewResolver** — компонент Spring MVC, преобразующий логическое имя представления, возвращённое [[@Controller]], в конкретную [[Web page]], готовую к рендерингу.

## Как это работает

Контроллер не возвращает HTML напрямую — он отдаёт короткое строковое имя: `return "users/list"`. `ViewResolver` добавляет к нему контекстные пути (префикс и суффикс) и формирует реальный путь к шаблону (например, `/WEB-INF/views/users/list.jsp`).

```java
public interface ViewResolver {
    View resolveViewName(String viewName, Locale locale) throws Exception;
}
```

`View` (`RenderingContext`) в свою очередь рендерит модель в HTML.

## Основные реализации

- **`InternalResourceViewResolver`** — для JSP-страниц. Типичная конфигурация: префикс `/WEB-INF/views/`, суффикс `.jsp`. Делегирует рендеринг в JSP-сервлет контейнера
- **`ThymeleafViewResolver`** — для Thymeleaf-шаблонов. Тесно интегрирован со Spring
- **`FreeMarkerViewResolver`**, **`VelocityViewResolver`** (устаревший)
- **`ContentNegotiatingViewResolver`** — выбирает View по типу контента (HTML, JSON, PDF — для одного URL возвращает разный формат)

## Bean-конфигурация

`ViewResolver` является [[@Bean]] в конфигурации [[RootConfigurer]]. Стандартная конфигурация:

### JSP

```java
@Bean
public ViewResolver viewResolver() {
    InternalResourceViewResolver resolver = new InternalResourceViewResolver();
    resolver.setPrefix("/WEB-INF/views/");
    resolver.setSuffix(".jsp");
    resolver.setViewClass(JstlView.class);
    return resolver;
}
// Controller:
@GetMapping("/users")
public String list(Model model) {
    model.addAttribute("users", service.findAll());
    return "users/list";   // → /WEB-INF/views/users/list.jsp
}
```

### Thymeleaf

```java
@Bean
public SpringResourceTemplateResolver templateResolver() {
    SpringResourceTemplateResolver r = new SpringResourceTemplateResolver();
    r.setPrefix("classpath:/templates/");
    r.setSuffix(".html");
    r.setTemplateMode(TemplateMode.HTML);
    return r;
}

@Bean
public SpringTemplateEngine templateEngine(SpringResourceTemplateResolver r) {
    SpringTemplateEngine e = new SpringTemplateEngine();
    e.setTemplateResolver(r);
    e.setEnableSpringELCompiler(true);
    return e;
}

@Bean
public ThymeleafViewResolver viewResolver(SpringTemplateEngine e) {
    ThymeleafViewResolver r = new ThymeleafViewResolver();
    r.setTemplateEngine(e);
    r.setCharacterEncoding("UTF-8");
    return r;
}
```

### Несколько ViewResolver'ов (порядок)

```java
@Bean
@Order(1)                        // приоритет
public ViewResolver jspResolver() { ... } // .jsp

@Bean
@Order(2)
public ViewResolver thymeleafResolver() { ... } // .html
```

### Для REST (`@RestController`) ViewResolver не нужен — `@ResponseBody`
сериализуется Jackson-ом напрямую в JSON/XML.

## Место в DispatcherServlet

`ViewResolver` работает в рамках [[DispatcherServlet]]: после того как [[Handler Mapping]] нашёл метод контроллера и [[Handler Adapter]] его исполнил, ViewResolver определяет, какую [[Web page]] отобразить.

```
[Controller returns "users/list"]
        ↓
[DispatcherServlet asks ViewResolver → View (Vel or Jsp)]
        ↓
[View.render(model, request, response) → HTML bytes]
        ↓
[Tomcat sends response]
```

friend:: [[Web page]]
friend:: [[RootConfigurer]]
friend:: [[DispatcherServlet]]
friend:: [[@Controller]]
friend:: [[@Bean]]
friend:: [[Handler Mapping]]
friend:: [[Handler Adapter]]