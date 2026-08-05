---
tags: [spring, annotation]
---

# @Configuration

**`@Configuration`** — аннотация Spring для Java-конфигурации. Является специализацией [[@Component]]. Класс, помеченный `@Configuration`, описывает бины декларативно через методы с [[@Bean]].

## CGLIB-проксирование

Spring обрабатывает `@Configuration`-классы через библиотеку **CGLIB**, создавая прокси поверх исходного класса. Это даёт два ключевых свойства:

- Методы [[@Bean]] вызываются контейнером единожды — гарантируется семантика singleton, даже если один `@Bean`-метод вызывается из другого напрямую (`someBean()`)
- Корректное разрешение межбиновых зависимостей внутри конфигурации

Без CGLIB (аналог `@Configuration(proxyBeanMethods = false)`) прямые вызовы `@Bean`-метода возвращали бы новые экземпляры — сокращённая конфигурация, используемая для экономии памяти и ускорения запуска.

## Использование

В классе, помеченном `@Configuration`, методы с аннотацией [[@Bean]] регистрируют бины в [[IoC Container]]. Дополнительно часто используется вместе с [[@ComponentScan]] для автоматической регистрации компонентов.

Используется в [[RootConfigurer]] Spring MVC приложения.

## Примеры

### Базовый пример

```java
@Configuration
public class AppConfig {

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    public DataSource dataSource() {
        return new HikariDataSource(hikariConfig());
    }

    // Кросс-ссылка между методами — благодаря CGLIB вернётся singleton
    @Bean
    public UserService userService() {
        return new UserService(passwordEncoder()); // тот же экземпляр, что в бине выше
    }
}
```

### Без proxyBeanMethods (быстрая конфигурация)

```java
@Configuration(proxyBeanMethods = false)
public class LiteConfig {
    @Bean
    public PasswordEncoder passwordEncoder() { return new BCryptPasswordEncoder(); }
    // Здесь нельзя вызывать passwordEncoder() напрямую — будет новый экземпляр
}
```

### С @ComponentScan

```java
@Configuration
@ComponentScan(basePackages = "com.example")
public class AppConfig { ... } // автoбнаружение @Service, @Repository и т.д.
```

### С @Import (модульная конфигурация)

```java
@Configuration
@Import({DbConfig.class, SecurityConfig.class})
public class AppConfig { ... }
```

## Другие аннотации конфигурационного класса

- `@ImportResource("classpath:beans.xml")` — смешать Java-конфигурацию с XML
- `@PropertySource("classpath:app.properties")` — подгрузить `.properties`
- `@Profile("dev")` — бины активны только в профиле

friend:: [[@Bean]]
friend:: [[RootConfigurer]]
friend:: [[@ComponentScan]]
friend:: [[IoC Container]]
parent:: [[@Component]]