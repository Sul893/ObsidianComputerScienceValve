---
tags: [spring, annotation]
---

# @Component

**`@Component`** — стереотипная аннотация Spring, обозначающая класс, управляемый [[ApplicationContext]] ([[IoC Container]]). Экземпляры таких классов (бины) создаются контейнером и регистрируются в реестре.

## Иерархия стереотипов

Другие аннотации являются её специализациями и автоматически наследуют поведение `@Component`:

- [[@Controller]] — для веб-слоя (Spring MVC), помечает классы-контроллеры
- [[@Configuration]] — для Java-конфигурации, классы содержат [[@Bean]]-методы
- `@Repository` — для слоя доступа к данным (DAO), добавляет конвертацию исключений (в `DataAccessException`)
- `@Service` — для бизнес-логики (маркер, без доп. поведения, но даёт семантическую ясность)

## Обнаружение

`@Component`-классы обнаруживаются через [[@ComponentScan]], который сканирует указанные пакеты. Классы без аннотаций Spring не находит.

## Жизненный цикл

Создаваемые бины попадают в контейнер [[IoC Container]] и управляются [[ApplicationContext]]: внедряются зависимости, вызываются lifecycle callback'и (`@PostConstruct`), контролируется скоуп (по умолчанию `singleton`).

## Пример

```java
@Component
public class EmailValidator {
    public boolean isValid(String email) {
        return email != null && email.contains("@");
    }
}
```

```java
@Service
public class UserService {
    private final EmailValidator validator;  // автоматическое внедрение
    public UserService(EmailValidator validator) { this.validator = validator; }
    public void register(String email) {
        if (!validator.isValid(email)) throw new IllegalArgumentException();
        // ...
    }
}
```

## Именование и скоуп

```java
@Component("primaryValidator")   // имя бина
@Scope("prototype")                // скоуп: новый экземпляр на каждый запрос
public class EmailValidator { ... }
```

`@Component`-классы могут реализовывать интерфейсы, и Spring при `@Autowired` выберет единственного кандидата; при нескольких — нужно `@Qualifier`.

## Иерархия в коде Spring

```java
@Target(TYPE) @Retention(RUNTIME)
public @interface Component {
    String value() default "";
}
@Repository  @Service  @Controller  @Configuration → @Component
```

friend:: [[@ComponentScan]]
friend:: [[@Controller]]
friend:: [[@Configuration]]
friend:: [[@Bean]]
friend:: [[IoC Container]]
parent:: [[ApplicationContext]]