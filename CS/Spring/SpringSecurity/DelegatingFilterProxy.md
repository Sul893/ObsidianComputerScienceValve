---
tags: [spring, security]
---

# DelegatingFilterProxy

**`DelegatingFilterProxy`** — сервлетный фильтр (`jakarta.servlet.Filter`) из Spring Framework, являющийся связующим звеном между [[Web Container]] ([[Apache Tomcat]]) и [[ApplicationContext]] Spring.

## Зачем нужен

Сервлетные фильтры регистрируются в `web.xml` (или через `ServletInitializer`) и живут в мире контейнера сервлетов — они ничего не знают о Spring-контексте и Spring-бинах Spring Security. `DelegatingFilterProxy` решает эту проблему: он сам регистрируется как фильтр контейнера, но делегирует всю реальную работу Spring-бину ([[FilterChainProxy]]), который ищется через [[ApplicationContext]].

## Контракт

```java
public class DelegatingFilterProxy extends GenericFilterBean {
    private String targetBeanName;   // имя бина, которому делегируется вызов

    @Override
    public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain) {
        Filter delegate = (Filter) findWebApplicationContext().getBean(targetBeanName);
        delegate.doFilter(req, resp, chain); // delegация!
    }
}
```

## Как работает

1. [[Apache Tomcat]] создаёт `DelegatingFilterProxy` при инициализации приложения
2. Все HTTP-запросы проходят сначала через него
3. `DelegatingFilterProxy` находит бин `springSecurityFilterChain` ([[FilterChainProxy]]) в Spring-контексте
4. Делегирует вызов `doFilter()` найденному бину

Таким образом, фильтрация выполняется Spring-кодом, но до [[DispatcherServlet]].

## Иерархия

`DelegatingFilterProxy` → делегирует → [[FilterChainProxy]] → цепочка [[SecurityFilterChain]] → [[Security Filter]].

## Регистрация

### web.xml (classic)

```xml
<filter>
    <filter-name>springSecurityFilterChain</filter-name>
    <filter-class>
        org.springframework.web.filter.DelegatingFilterProxy
    </filter-class>
</filter>
<filter-mapping>
    <filter-name>springSecurityFilterChain</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

### Программно (Servlet 3+, Spring Boot)

```java
public class SecurityWebAppInitializer
        extends AbstractSecurityWebApplicationInitializer {
    // Регистрирует DelegatingFilterProxy как сервлетный фильтр, 
    // который ищет бин "springSecurityFilterChain" в ApplicationContext.
    // Тело класса может быть пустым — базовый класс делает всю работу.
}
```

В Spring Boot это автоматически делается `SecurityAutoConfiguration` — ручная регистрация не нужна.

## Минимальный полный пример

```java
// 1. Настройка безопасности
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.authorizeHttpRequests(a -> a.anyRequest().authenticated())
            .httpBasic(Customizer.withDefaults());
        return http.build();
    }
}

// 2. Spring Boot автоматически регистрирует DelegatingFilterProxy
// → он делегирует в FilterChainProxy → UserDetailsService, auth-фильтры, ...
```

friend:: [[ServletContext]]
friend:: [[ApplicationContext]]
friend:: [[Web Container]]
friend:: [[FilterChainProxy]]
parent:: [[Servlet filter]]