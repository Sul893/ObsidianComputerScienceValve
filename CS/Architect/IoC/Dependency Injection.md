---
tags: [architect]
---
#### Dependency Injection

> "Its giving an object its variables"

Внедрение зависимотей - это архитектурный паттерн проектирования при котором зависимости объекта предаются извне.

Представим классическую ситуацию:
```java
class MyDAO{
	DataSource ds = new DataSourceImpl(name="Postgres", location:"localhost:5432", user:"me", password:"1234")
}
```
Поскольку DAO использует данные переменных для создания экземпляра DataSourceImpl, то он зависит от их значений. 

Теперь каждый класс DAO будет зависеть от одного источника и когда мы будем тестировать, придется прописывать новый класс с новым источником/ользователем.

Чтобы упростить себе жизнь и обобщить создание DataSourceImpl можно вынести переменные в конструктор/сеттер и определять их значения во внешнем классе:

```java
class MyDAO{
	DataSource ds;
	public MyDAO(name, location, user, password){
		this.ds = new DataSourceImpl(name, location, user, password)
	}
}
```

Теперь наш класс не зависит от переменных, необходимых для создания DataSourceImpl

можно пойти дальше и использовать [[Dependency Inversion Principle]] чтобы не зависеть от конкретной реализации и использовать интерфейс DataSource

```java
class MyDAO{
	DataSource ds;
	public MyDAO(DataSource ds){
		this.ds = ds
	}
	
	...
	
}
```

Но кто теперь отвечает за наши зависимости?

```java
class MyComponent{
    MyDao dao = new MyDao(new DataSourceImpl("driver", "url", "user", "password"));
       
	...
	
}
```

И теперь наша компонента отвечает еще и за класс DAO и будет знать про конкректную инфорацию о источнике данных, что еще хуже чем было.
Что делать?
Пробрасывать дальше

И мы так и будет пробрасывать вверх по дереву, пока не дойдем до... чего?

Вообще как можно было догадаться, DI - это реализация [[IoC]] и для его работы необходим какой-то финальный корневой класс, в котором расписаны все зависимости - [[Composition Root]]


![[IoC2.excalidraw]]

(Слева граф зависимостей - справа использование)

В [[Composition Root]] определяются все связи, внедряются все зависимости и происходит общая инициализация.
Фреймворки по-разному его реализуют. вот самая примитивная реализация:

```java
class Program
{
    static void Main(string[] args)
    {
        // Composition Root начинается здесь
        var connectionString = Configuration.GetConnectionString("Default");
        var repository = new SqlUserRepository(connectionString);
        var userService = new UserService(repository);
        var controller = new HomeController(userService);
        
        // Запускаем приложение, передавая готовый граф
        var app = new ConsoleApp(controller);
        app.Run();
    }
}
```

parent::[[IoC]]
friend::[[Composition Root]]
friend::[[Dependency Inversion Principle]]
