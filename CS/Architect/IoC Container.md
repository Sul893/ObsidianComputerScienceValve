---
tags: [architect]
---

# IoC Container

**IoC Container** — контейнер, реализующий принцип [[IoC]] (в частности [[Dependency Inversion Principle|DIP]]). Управляет созданием объектов, их жизненным циклом и внедрением зависимостей через [[DI]].

## Основные функции

- **Создание и хранение бинов** — контейнер создаёт объекты по описанию (метаданные/аннотации/XML) и хранит их в реестре.
- **Внедрение зависимостей** — автоматически связывает бины между собой, разрешая ссылки по типу или имени.
- **Управление жизненным циклом** — вызывает `init`/`destroy` callback'и, применяет пост-обработку.
- **Скоупы (области видимости)** — определяет, сколько экземпляров бина создаётся:
  - `singleton` — один бин на контейнер (по умолчанию)
  - `prototype` — новый экземпляр на каждый запрос
  - `request` — один бин на HTTP-запрос (web)
  - `session` — один бин на HTTP-сессию (web)
- **Конфигурирование** — бины описываются декларативно (аннотации/метаданные/XML).

## Как это работает

Контейнер при старте приложения:
1. Читает конфигурацию (метаданные/аннотации/XML)
2. Создаёт экземпляры бинов (через рефлексию)
3. Разрешает и внедряет зависимости (через конструктор, setter, поле)
4. Вызывает lifecycle-callback'и (`init()` после создания, `destroy()` перед выгрузкой)
5. Хранит бины в реестре и предоставляет их по запросу

## Упрощённая модель работы контейнера

```java
// Контейнер в упрощённом виде:
class SimpleContainer {
    Map<Class<?>, Object> beans = new HashMap<>();

    <T> T getBean(Class<T> type) {
        return (T) beans.get(type);
    }

    <T> void register(Class<T> type, Supplier<T> supplier) {
        beans.put(type, supplier.get());
    }
}

// Использование:
SimpleContainer ctx = new SimpleContainer();
ctx.register(UserRepository.class, () -> new JdbcUserRepository(dataSource));
ctx.register(UserService.class, () -> new UserService(ctx.getBean(UserRepository.class)));

UserService service = ctx.getBean(UserService.class);
```

## Скоупы (области видимости)

Скоуп указывается в описании бина:

```java
// singleton (по умолчанию) — один экземпляр на контейнер
container.register(SingletonBean.class, SingletonBean::new);

// prototype — новый экземпляр при каждом запросе
container.registerPrototype(PrototypeBean.class, PrototypeBean::new);

// request-context — один бин на HTTP-запрос (только в web-контейнере)
container.registerScoped("request", RequestBean.class, RequestBean::new);
```

## Lifecycle-callback'и

Контейнер оповещает бин о ключевых точках жизненного цикла. Типичные контракты — `init`/`destroy` или интерфейсы с аналогичными методами:

```java
public class LifecycleDemo implements Initializable, Disposable {

    // init — вызывается после внедрения зависимостей
    @Override
    public void afterPropertiesSet() {
        System.out.println("Init: зависимости готовы");
    }

    // destroy — вызывается перед уничтожением бина (например, при закрытии контекста)
    @Override
    public void destroy() {
        System.out.println("Destroy: освобождаем ресурсы");
    }
}

// Альтернатива —命名-конвенция: контейнер вызывает методы по имени
public class LifecycleDemo {
    public void init()  { /* имя метода указано в описании бина */ }
    public void destroy() { /* имя метода указано в описании бина */ }
}
```

> `Initializable` / `Disposable` — имена для примера. В конкретных фреймворках эти интерфейсы могут называться иначе (например, через аннотацию `@PostConstruct`/`@PreDestroy` из Jakarta EE или через naming-конвенцию `init-method`/`destroy-method`), но контракт тот же: "init после внедрения — destroy перед выгрузкой".

## Базовый и расширенный контейнер

IoC-фреймворки обычно предоставляют два уровня контейнера:

- **Базовый** (`BeanFactory`-стиль) — конфигурация и управление бинами, ленивая инициализация.
- **Расширенный** — добавляет поверх базового: события (publish/subscribe), интернационализацию, загрузку ресурсов, автоматическую регистрацию post-processor'ов, eager-инициализацию singleton'ов (вместо ленивой).

parent:: [[DI]]
friend:: [[Dependency Inversion Principle]]
friend:: [[IoC]]