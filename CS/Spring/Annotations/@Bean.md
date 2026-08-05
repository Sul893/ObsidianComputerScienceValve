---
tags: [spring, annotation]
---

# @Bean

**`@Bean`** — аннотация Spring для объявления бина в Java-конфигурации. Используется внутри класса, помеченного [[@Configuration]] (или `@Component`).

## Суть

Метод с `@Bean` возвращает объект, который Spring регистрирует в [[IoC Container]] ([[ApplicationContext]]) как бин. Вызов метода берёт на себя контейнер, а не прикладной код.

В отличие от [[@Component]] (где бин = сам класс), `@Bean` позволяет регистрировать сторонние классы, которые нельзя аннотировать (библиотечные), и явно управлять процессом создания.

## Ключевые атрибуты

- `name` / `value` — имя бина (по умолчанию совпадает с именем метода)
- `initMethod` — метод объекта, вызываемый после создания (альтернатива `@PostConstruct`)
- `destroyMethod` — метод, вызываемый при уничтожении бина (альтернатива `@PreDestroy`)
- `autowireCandidate` — может ли бин быть кандидатом для `@Autowired`
- `scope` (через `@Scope`) — область видимости (`singleton`/`prototype`/...)
- `primary` (через `@Primary`) — предпочесть этот бин при множественных кандидатах
- `lazy` (через `@Lazy`) — отложенная инициализация

## По умолчанию

- **Скоуп**: `singleton` (один экземпляр); меняется через `@Scope`
- **Имя**: = имени метода

## Примеры

### Базовый пример

```java
@Configuration
public class AppConfig {

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

### Имя бина, скоуп, lifecycle

```java
@Bean(name = "primaryPwdEncoder")
@Scope("singleton")
@Primary
public PasswordEncoder passwordEncoder() {
    BCryptPasswordEncoder encoder = new BCryptPasswordEncoder(12);
    return encoder;
}

@Bean(initMethod = "init", destroyMethod = "close")
public DataSource dataSource() {
    HikariDataSource ds = new HikariDataSource();
    ds.setJdbcUrl("jdbc:postgresql://localhost/myapp");
    ds.setUsername("app");
    ds.setPassword("secret");
    return ds;
}
```

### Внедрение зависимостей в @Bean-методы

```java
@Bean
public UserService userService(UserRepository repo,
                               PasswordEncoder encoder) {
    // Spring автоматически подставит бины по типам параметров
    return new UserService(repo, encoder);
}
```

### Регистрация объекта состояния (builder pattern)

```java
@Bean
public WebClient webClient(@Value("${api.base-url}") String baseUrl) {
    return WebClient.builder()
                    .baseUrl(baseUrl)
                    .build();
}
```

## Когда использовать @Bean vs @Component

| Случай | Аннотация |
|--------|----------|
| Свой класс, можно аннотировать | `@Component` |
| Сторонняя библиотека (DataSource, WebClient) | `@Bean` в `@Configuration` |
| Несколько бинов одного типа с разными настройками | `@Bean` |

parent:: [[@Configuration]]
friend:: [[@Component]]
friend:: [[IoC Container]]
friend:: [[ApplicationContext]]