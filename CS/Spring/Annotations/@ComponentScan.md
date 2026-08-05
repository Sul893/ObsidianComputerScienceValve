---
tags: [spring, annotation]
---

# @ComponentScan

**`@ComponentScan`** — аннотация Spring, указывающая пакеты, которые контейнер должен отсканировать для нахождения классов, помеченных [[@Component]] и его специализациями (@Controller, @Service, @Repository, [[@Configuration]]).

## Параметры

- `basePackages` — список пакетов для сканирования (например: `basePackages = {"com.app.services"}`)
- `basePackageClasses` — классы-маркеры, чьи пакеты будут отсканированы (типобезопасный вариант: refactoring-move package не сломает сканирование)
- `excludeFilters` / `includeFilters` — фильтры для исключения/включения классов из сканирования по аннотациям, типам, регуляркам
- `nameGenerator` — стратегия именования бинов
- `scopeResolver` — стратегия разрешения скоупа
- `lazyInit` — признак отложенной инициализации найденных бинов (по умолчанию `false`)
- `useDefaultFilters` — включить дефолтные фильтры (`@Component`, `@Repository`, ...), по умолчанию `true`

## Поведение по умолчанию

Если параметр `basePackages`/`basePackageClasses` не указан, сканируется пакет, в котором находится класс с аннотацией [[@Configuration]] — все его подпакеты.

## Минимальный пример

```java
@Configuration
@ComponentScan(basePackages = "com.example")
public class AppConfig {}
```

## С фильтрами

```java
@Configuration
@ComponentScan(
    basePackages = "com.example",
    excludeFilters = @ComponentScan.Filter(
        type = FilterType.REGEX,
        pattern = "com\\.example\\.legacy\\..*"
    ),
    includeFilters = @ComponentScan.Filter(
        type = FilterType.ANNOTATION,
        classes = Marker.class
    )
)
public class AppConfig {}
```

## Несколько пакетов

```java
@ComponentScan(basePackages = {
    "com.example.services",
    "com.example.repositories"
})
```

Обычно размещается в [[RootConfigurer]], который содержится внутри [[ApplicationContext]] ([[Root WebApplicationContext]]).

## В Spring Boot

В Spring Boot `@ComponentScan` не нужен явно — `@SpringBootApplication` включает его (по умолчанию сканируется пакет, в котором находится main-класс).

```java
@SpringBootApplication
// Эквивалентно:
// @Configuration @EnableAutoConfiguration @ComponentScan(basePackages="com.example.app")
public class MyApplication { ... }
```

friend:: [[@Component]]
friend:: [[RootConfigurer]]
friend:: [[ApplicationContext]]
friend:: [[Root WebApplicationContext]]
parent:: [[@Configuration]]