---
tags: [architect]
---

# GRASP

**GRASP (General Responsibility Assignment Software Patterns)** — набор паттернов для распределения обязанностей между классами в объектно-ориентированном проектировании. Описаны Крейгом Ларманом.

В отличие от паттернов GoF, GRASP не описывает конкретную структуру классов, а отвечает на вопрос: *"Какие обязанности и какому классу следует назначить?"*

## Паттерны

### Information Expert

Обязанность назначается классу, владеющему нужными для её выполнения данными. Снижает количество "пересылок" данных между объектами.

```java
// Information Expert — класс Order сам считает сумму, т.к. владеет items
class Order {
    private List<Item> items;
    public BigDecimal total() {
        return items.stream().map(Item::price).reduce(ZERO, BigDecimal::add);
    }
}
// НЕ: OrderService.total(order) — он вынужден вытащить items из order
```

### Creator

Класс `B` должен создавать объекты класса `A`, если `B` агрегирует/содержит `A`, использует `A` или владеет данными для инициализации `A`.

```java
// Order содержит OrderItems → Order создаёт их
class Order {
    private List<OrderItem> items = new ArrayList<>();
    public void addItem(Product p, int qty) {
        items.add(new OrderItem(p, qty)); // Order — Creator для OrderItem
    }
}
```

### Controller

Выделенный класс-посредник между UI и бизнес-логикой. Принимает входные события и делегирует их в предметную область, не содержа сам бизнес-логику.

```java
// Controller — посредник между UI и доменом, не содержит бизнес-логику
public class OrderController {
    private final OrderService service; // делегирует в домен
    public void onCreate(OrderForm form) {
        service.create(form); // бизнес-логика — в OrderService
    }
}
```

### Low Coupling

Минимизация связанности между классами. Чем меньше класс знает о других, тем проще его изменять и переиспользовать.

```java
// Высокая связанность
class Report {
    private JdbcDataSource ds = new JdbcDataSource(); // прямая зависимость
}

// Низкая связанность — через абстракцию
class Report {
    private final DataSource ds;
    Report(DataSource ds) { this.ds = ds; } // подставим любую реализацию
}
```

### High Cohesion

Класс должен быть сфокусирован на одной задаче. Высокая сплочённость облегчает понимание и переиспользование.

```java
// Низкая сплочённость — один класс делает всё (God Object)
class GodService {
    void createUser() { ... }
    void sendEmail() { ... }
    void generateReport() { ... }
}

// Высокая сплочённость — разделение
class UserService { void createUser() { ... } }
class EmailService { void sendEmail() { ... } }
class ReportService { void generateReport() { ... } }
```

### Polymorphism

Вариативность поведения реализуется через полиморфизм (интерфейсы/наследование), а не через `if/switch` по типу.

```java
// Нарушение
double area(Shape s) {
    if (s instanceof Circle) return Math.PI * r * r;
    else if (s instanceof Square) return side * side;
}

// Соблюдение
interface Shape { double area(); }
class Circle implements Shape { public double area() { ... } }
```

### Pure Fabrication

Искусственный класс, не имеющий аналога в предметной области, вводимый для снижения связанности (например, `Repository` или `Mapper`).

```java
// PaymentRepository — нет в домене, добавлен ради separation of concerns
public interface PaymentRepository { // Pure Fabrication
    void save(Payment p);
}
public class JdbcPaymentRepository implements PaymentRepository { ... }
```

### Indirection

Введение промежуточного слоя для снижения связанности между компонентами (например, `Service` между `Controller` и `Repository`).

```java
// Indirection — Service изолирует Controller от Repository и Domain Logic
class OrderController {
    private final OrderService service; // слой Indirection
}
class OrderService {
    private final OrderRepository repo; // ещё один слой Indirection
}
```

### Protected Variations

Защита от изменений через интерфейсы и абстракции: точки нестабильности скрываются за стабильной абстракцией.

```java
// Protected Variations — стабильный интерфейс PaymentGateway
// скрывает вариации провайдеров (Stripe, PayPal)
interface PaymentGateway { void charge(Payment p); }

class StripeGateway implements PaymentGateway { ... }
class PayPalGateway implements PaymentGateway { ... }
// Смена провайдера не затрагивает вызывающий код
```

## Связь с другими парадигмами

GRASP тесно связано с принципами [[SOLID]] (особенно SRP ↔ High Cohesion, Low Coupling ↔ OCP/DIP) и находит практическое применение в [[DI]].

friend:: [[SOLID]]
friend:: [[DI]]