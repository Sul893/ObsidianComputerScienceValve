---
tags: [architect]
---

# Service Locator

**Service Locator** — паттерн проектирования, реализующий [[Dependency Inversion Principle]] по pull-модели: объекты сами запрашивают свои зависимости у центрального локатора служб.

В отличие от [[DI]] (push-модель), где контейнер сам внедряет зависимости, при Service Locator класс обращается к локатору: `paymentRepository = ServiceLocator.get(PaymentRepository.class)`.

## Как это работает

1. При старте приложения регистрируются реализации в `ServiceLocator`.
2. Класс, которому нужна зависимость, обращается к локатору в момент вызова метода или инициализации.
3. Локатор возвращает готовый экземпляр или создаёт его по требованию.

## Пример реализации

```java
// Простой Service Locator
public class ServiceLocator {
    private static final Map<Class<?>, Object> registry = new ConcurrentHashMap<>();

    public static <T> void register(Class<T> type, T instance) {
        registry.put(type, instance);
    }

    @SuppressWarnings("unchecked")
    public static <T> T get(Class<T> type) {
        return (T) registry.get(type);
    }
}

// При старте приложения
ServiceLocator.register(PaymentRepository.class, new JdbcPaymentRepository());

// Использование в классе
class OrderService {
    public void process(Order order) {
        PaymentRepository repo = ServiceLocator.get(PaymentRepository.class);
        repo.save(order);
    }
}
```

## Сравнение с DI

| Критерий | DI | Service Locator |
|----------|----|-----------------|
| Модель | push (внедрение извне) | pull (запрос самим объектом) |
| Видимость зависимостей | в конструкторе/сигнатуре | скрыты внутри метода |
| Тестирование | передаём мок в конструктор | нужно мокать локатор |
| Ошибки | на старте (при сборке контекста) | в рантайме |
| Считается | хорошей практикой | антипаттерном |

## Почему считается антипаттерном

- Зависимости класса не видны в его интерфейсе — нельзя определить требования по конструктору, приходится читать реализацию.
- Усложняется модульное тестирование: вместо мока конкретной зависимости приходится мокать сам `ServiceLocator`, что загрязняет тесты.
- Создаёт неявную зависимость от инфраструктурного компонента (локатора).
- Возможны ошибки времени выполнения: запрошенная зависимость не зарегистрирована, что компилятор не отловит.

Основан на принципе [[IoC]].

parent:: [[Dependency Inversion Principle]]
friend:: [[DI]]
friend:: [[IoC]]