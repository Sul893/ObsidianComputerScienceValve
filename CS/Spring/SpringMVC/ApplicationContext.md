---
tags: [spring, springmvc]
---

# ApplicationContext

**`ApplicationContext`** — интерфейс Spring, реализация [[IoC Container]]. Центральный механизм DI: создаёт, связывает и управляет бинами. Расширяет интерфейс `BeanFactory`, добавляя: поддержку [скоупов](IoC%20Container.md#Скоупы%20(области%20видимости)), событий (`ApplicationEvent`), интернационализации (`MessageSource`), загрузки ресурсов (`ResourceLoader`) и аннотаций `@PostConstruct`, `@Autowired`.

## Иерархия интерфейсов

```
BeanFactory (base) ──  ListableBeanFactory, HierarchicalBeanFactory
                                 │
                       ApplicationContext   ← добавляет события, i18n, ...
```

Основные concrete-реализации:
- `ClassPathXmlApplicationContext` (XML из classpath)
- `AnnotationConfigApplicationContext` (Java-конфигурация)
- `GenericWebApplicationContext` (web)
- `AnnotationConfigServletWebServerApplicationContext` (Spring Boot, embedded Tomcat)

## Иерархия контекстов в веб-приложении

Spring MVC использует двухуровневую иерархию контекстов:

- **[[Root WebApplicationContext]]** — родительский контекст. Содержит сервисы, репозитории, инфраструктуру, общий для всего приложения.
- **[[Servlet WebApplicationContext]]** — дочерний контекст на каждый [[DispatcherServlet]]. Содержит веб-компоненты: контроллеры, handler mappings, view resolvers. Может обращаться к бинам родителя, но не наоборот.

Создаётся через [[ContextLoaderListener]] при старте [[Apache Tomcat]] ([[Web Container]]).

## Связь с ServletContext

`ApplicationContext` связан с миром сервлетов через [[ServletContext]]: корневый контекст помещается в ServletContext как атрибут, чтобы другие компоненты могли его получить.

## Lifecycle Bean (хронология)

```
ApplicationContext.refresh()
  ↓
BeanDefinitionReader читает @Configuration/@ComponentScan
  ↓
BeanPostProcessor.before
  ↓
@Bean-instantiation (через рефлексию)
  ↓
внедрение зависимостей (@Autowired или constructor)
  ↓
  @PostConstruct / InitializingBean#afterPropertiesSet
  ↓
  BeanPostProcessor.after (проксирование для AOP)
  ↓
бины готовы к использованию
  ↓
context#close() → @PreDestroy / DisposableBean#destroy
```

## События (ApplicationEvent)

```java
@Component
public class UserCreatedListener {
    @EventListener
    public void onUserCreated(UserCreated event) {
        // срабатывает на event.applicationContext.publishEvent(new UserCreated(...))
        System.out.println("User created: " + event.getUsername());
    }
}
```

## Доступ к ApplicationContext из кода

```java
@Autowired
private ApplicationContext ctx;                        // внедряется по типу

ctx.getBean(UserService.class);
ctx.getBean("primaryEncoder", PasswordEncoder.class); // по имени
ctx.getEnvironment().getProperty("app.name");          // доступ к properties
ctx.publishEvent(new UserCreated("alice"));            // публикация события
```

friend:: [[ServletContext]]
friend:: [[ContextLoaderListener]]
friend:: [[Root WebApplicationContext]]
friend:: [[Servlet WebApplicationContext]]
friend:: [[DispatcherServlet]]
friend:: [[Apache Tomcat]]
friend:: [[Web Container]]
parent:: [[IoC Container]]