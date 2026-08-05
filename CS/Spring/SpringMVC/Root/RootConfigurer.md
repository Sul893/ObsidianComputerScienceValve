---
tags: [spring, springmvc]
---

# RootConfigurer

**RootConfigurer** — главный конфигурационный класс корневого контекста. [[Root WebApplicationContext]] (реализация [[ApplicationContext]]) создаётся на основе него и содержит его.

## Что содержит

- Аннотацию [[@Configuration]] — маркирует класс как конфигурацию Spring
- Аннотацию [[@ComponentScan]] — для автообнаружения [[@Component]] (сервисов, репозиториев)
- [[@Bean]]-методы — для явной регистрации бинов, например [[ViewResolver]]
- Опционально `@Import`, `@PropertySource`, `@EnableTransactionManagement`

## Пример для Spring MVC webapp

```java
@Configuration
@ComponentScan(basePackages = "com.example")
@EnableTransactionManagement
@PropertySource("classpath:application.properties")
public class RootConfigurer {

    @Bean
    public DataSource dataSource(@Value("${db.url}") String url) {
        HikariConfig cfg = new HikariConfig();
        cfg.setJdbcUrl(url);
        cfg.setUsername("app");
        return new HikariDataSource(cfg);
    }

    @Bean
    public PlatformTransactionManager txManager(DataSource ds) {
        return new DataSourceTransactionManager(ds);
    }

    @Bean
    public ViewResolver viewResolver() {
        InternalResourceViewResolver r = new InternalResourceViewResolver();
        r.setPrefix("/WEB-INF/views/");
        r.setSuffix(".jsp");
        r.setViewClass(JstlView.class);
        return r;
    }

    @Bean
    public MessageSource messageSource() {
        ReloadableResourceBundleMessageSource m =
            new ReloadableResourceBundleMessageSource();
        m.setBasename("classpath:i18n/messages");
        m.setDefaultEncoding("UTF-8");
        return m;
    }
}
```

## Регистрация RootConfigurer

### Через web.xml (classic)

```xml
<context-param>
    <param-name>contextClass</param-name>
    <param-value>
        org.springframework.web.context.support.AnnotationConfigWebApplicationContext
    </param-value>
</context-param>
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>com.example.RootConfigurer</param-value>
</context-param>

<listener>
    <listener-class>
        org.springframework.web.context.ContextLoaderListener
    </listener-class>
</listener>
```

### Программно (Servlet 3 / Spring Boot)

```java
public class WebInitializer implements WebApplicationInitializer {
    public void onStartup(ServletContext container) {
        AnnotationConfigWebApplicationContext root = new AnnotationConfigWebApplicationContext();
        root.register(RootConfigurer.class);
        container.addListener(new ContextLoaderListener(root));
    }
}
```

## В Spring Boot

Эквивалент — класс с `@SpringBootApplication`. Он объединяет Root и Servlet-конфигурацию в один контекст Spring Boot, и RootConfigurer явно не нужен.

```java
@SpringBootApplication
public class SpringBootApp {
    public static void main(String[] args) {
        SpringApplication.run(SpringBootApp.class, args);
    }
}
```

friend:: [[@ComponentScan]]
friend:: [[@Configuration]]
friend:: [[@Bean]]
friend:: [[ViewResolver]]
friend:: [[Root WebApplicationContext]]
friend:: [[ApplicationContext]]
friend:: [[@Component]]