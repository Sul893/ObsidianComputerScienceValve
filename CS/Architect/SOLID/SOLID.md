---
tags: [architect, solid]
---

# SOLID

**SOLID** — пять принципов объектно-ориентированного проектирования, сформулированных Робертом Мартином, для создания поддерживаемого, расширяемого и тестируемого кода.

## Принципы

### S — Single Responsibility Principle (SRP)

У каждого класса должна быть только одна причина для изменения. Класс отвечает за одну задачу и должен изменяться только при изменении этой задачи.

```java
// Нарушение SRP: класс одновременно считает и печатает отчёт
class Report {
    void calculate() { ... }
    void printPdf() { ... }   // лишняя ответственность
}

// Соблюдение SRP: разделение
class Report { void calculate() { ... } }
class ReportPrinter { void printPdf(Report r) { ... } }
```

### O — Open/Closed Principle (OCP)

Программные сущности должны быть открыты для расширения, но закрыты для модификации. Новое поведение добавляется через наследование или композицию, а не изменением существующего кода.

```java
// Нарушение OCP: при добавлении нового типа — редактируем switch
double area(Shape s) {
    if (s instanceof Circle) return Math.PI * r * r;
    else if (s instanceof Square) return side * side;
}

// Соблюдение OCP: полиморфизм, классы сами реализуют area()
interface Shape { double area(); }
class Circle implements Shape { public double area() { ... } }
// Новый тип: class Triangle implements Shape {...} — без изменения area()
```

### L — Liskov Substitution Principle (LSP)

Подтипы должны быть заменяемыми базовыми типами без нарушения корректности программы. Поведение наследника не должно противоречить контракту родителя.

```java
// Нарушение LSP: Square ломает контракт Rectangle
class Rectangle { void setWidth(int w){} void setHeight(int h){} }
class Square extends Rectangle {
    void setWidth(int w){ super.setWidth(w); super.setHeight(w); } // нарушает ожидания
}

// Соблюдение: общий интерфейс
interface Shape { double area(); }
```

### I — Interface Segregation Principle (ISP)

Лучше несколько специализированных интерфейсов, чем один универсальный. Клиенты не должны зависеть от методов, которые они не используют.

```java
// Нарушение ISP: один жирный интерфейс
interface Worker { void work(); void eat(); }
class Robot implements Worker {
    public void work() { ... }
    public void eat() { throw new UnsupportedOperationException(); } // не нужно
}

// Соблюдение ISP: разделение
interface Workable { void work(); }
interface Eatable { void eat(); }
class Robot implements Workable { public void work() { ... } }
class Human implements Workable, Eatable { ... }
```

### D — Dependency Inversion Principle (DIP)

Модули верхнего уровня не должны зависеть от модулей нижнего уровня; оба должны зависеть от абстракций. См. [[Dependency Inversion Principle]].

```java
// Нарушение DIP
class OrderService {
    private final JdbcPaymentRepository repo = new JdbcPaymentRepository();
}

// Соблюдение DIP
class OrderService {
    private final PaymentRepository repo; // абстракция
    OrderService(PaymentRepository repo) { this.repo = repo; }
}
```

## Связь с другими парадигмами

Принципы SOLID дополняются паттернами [[GRASP]] (общие паттерны назначения обязанностей).

friend:: [[GRASP]]
friend:: [[Dependency Inversion Principle]]