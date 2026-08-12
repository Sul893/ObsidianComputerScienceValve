---
tags: [architect, solid]
---

# Dependency Inversion Principle

**Принцип инверсии зависимостей (DIP)** - модули верхнего уровня не должны зависеть от модулей нижнего уровня. Оба должны зависеть от абстракций. Абстракции не должны зависеть от деталей; детали должны зависеть от абстракций.

Является буквой **D** в акрониме [[SOLID]] и частным случаем более общего принципа [[IoC]].

## Суть

В традиционном (нарушающем DIP) дизайне высокоуровневый компонент (например, `OrderService`) напрямую создаёт и использует низкоуровневый компонент (`PaymentRepository`). Это приводит к сильной связанности: любое изменение низкого уровня затрагивает высокий.

DIP требует, чтобы:
1. Высокий уровень зависел от абстракции (`PaymentRepository` interface), а не от конкретного класса.
2. Низкий уровень реализовывал эту абстракцию (`JdbcPaymentRepository implements PaymentRepository`).
3. Связывание происходило извне, через конструктор или setter.

## Пример

```java
// Нарушение DIP: высокий уровень напрямую зависит от низкого
class OrderService {
    private final JdbcPaymentRepository repo = new JdbcPaymentRepository();
    // Любая смена реализации → изменение OrderService
}

// Соблюдение DIP: оба зависят от абстракции
interface PaymentRepository {              // абстракция
    void save(Order order);
}

class JdbcPaymentRepository                // деталь (низкий уровень)
        implements PaymentRepository {
    public void save(Order order) { /* JDBC */ }
}

class OrderService {                       // высокий уровень
    private final PaymentRepository repo;  // зависимость от абстракции
    OrderService(PaymentRepository repo) { this.repo = repo; }
}

// Связывание происходит извне:
PaymentRepository repo = new JdbcPaymentRepository();
OrderService service = new OrderService(repo);
```

## Реализации

Принцип DIP реализуется через:
- [[Dependency Injection]] — внедрение зависимостей извне (рекомендуемый подход)
- [[Service Locator]] — объект сам запрашивает зависимости (считается антипаттерном)

parent:: [[IoC]]
parent:: [[SOLID]]
friend:: [[Dependency Injection]]
friend:: [[Service Locator]]