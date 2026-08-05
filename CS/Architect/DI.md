---
tags: [architect]
---

# Dependency Injection

**Dependency Injection (DI)** — паттерн, реализующий [[Dependency Inversion Principle]]. Зависимости внедряются в объект извне, вместо того чтобы объект создавал их сам.

Управляется [[IoC Container]], который отвечает за создание, связывание и жизненный цикл объектов.

## Суть

Вместо того чтобы класс `OrderService` сам создавал `PaymentRepository` через `new`, контейнер внедряет готовый экземпляр `PaymentRepository` в `OrderService`. Класс декларирует зависимость, а контейнер её удовлетворяет.

### Без DI (нарушение DIP)

```java
class OrderService {
    // Класс сам создаёт конкретную реализацию — сильная связанность
    private final JdbcPaymentRepository repo = new JdbcPaymentRepository();

    public void process(Order order) {
        repo.save(order);
    }
}
```

### С DI

```java
class OrderService {
    private final PaymentRepository repo; // зависит от абстракции

    // Зависимость внедряется через конструктор — push-модель
    OrderService(PaymentRepository repo) {
        this.repo = repo;
    }

    public void process(Order order) {
        repo.save(order);
    }
}
```

## Способы внедрения

### Constructor Injection (рекомендуется)

Зависимость передаётся через конструктор. Гарантирует, что объект всегда валиден и поле может быть `final`.

```java
class UserService {
    private final UserRepository userRepository;
    private final PasswordValidator passwordValidator;

    UserService(UserRepository userRepository, PasswordValidator passwordValidator) {
        this.userRepository = userRepository;
        this.passwordValidator = passwordValidator;
    }
}
```

Плюсы: зависимости видны в сигнатуре конструктора, неизменяемость (`final`), удобство тестирования.
Минусы: при большом количестве зависимостей конструктор разрастается (сигнал о нарушении SRP).

### Setter Injection

Зависимость передаётся через setter-метод. Подходит для опциональных зависимостей.

```java
class ReportService {
    private OptionalFormatter formatter;

    public void setFormatter(OptionalFormatter formatter) {
        this.formatter = formatter;
    }
}
```

Плюсы: возможность переназначения, опциональность.
Минусы: объект может оказаться в невалидном состоянии (setter не вызвали), нельзя `final`.

### Field Injection (не рекомендуется)

Внедрение напрямую в поле (через reflection или IoC-специфичные аннотации).

```java
class UserService {
    private UserRepository userRepository;  // не final, скрытая зависимость
    // контейнер устанавливает поле через reflection
}
```

Плюсы: краткий синтаксис.
Минусы: скрывает зависимости, усложняет тестирование, нельзя `final`.

## Разрешение зависимостей по типу/имени

При наличии нескольких реализаций одного интерфейса контейнер выбирает по имени или маркеру:

```java
// Селекция по маркеру
class PaymentService {
    PaymentService(PaymentGateway paypal, PaymentGateway stripe) { ... }
    // контейнер различает по имени параметра или по @Qualifier-аналогу
}
```

## Альтернатива

Альтернативный подход к разрешению зависимостей — [[Service Locator]] (pull-модель), где объект сам запрашивает зависимости у локатора. В отличие от DI (push-модель), Service Locator считается антипаттерном.

parent:: [[Dependency Inversion Principle]]
friend:: [[Service Locator]]
friend:: [[IoC Container]]