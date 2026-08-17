---
tags: [architect]
---

#### Service Locator

В современной разработке считается анти-паттерном, но наверное и ему есть применение.

Суть в том, что создается библиотека сервайс-объектов. Клиенты (объекты вызывающие у себя сервайсы) инициализируют зависимости у себя, Service Locator используется просто как поисковик.

Теперь чуть подробнее:

У нас есть:
- client  - т.е класс, вызывающий у себя локатор для поиска
- cache - хранилище объектов сервайсов
- Service Locator - возвращает объекты как только его просят
- Initial context - создание объектов сервайсов
- Service - храимый объект класса

Предположим что есть какой-то MessagingInterface

```java
public interface MessagingInterface{
	public void sendImage();
	public void sendText();
}
```

И его имплементации

```java
public class MakeSMS implements MessagingInterface{
	// ...
}
```
```java
public class MakeEmail implements MessagingInterface{
	// ...
}
```

надо их где-то хранить, создадим класс Cache с list-ом

```java

public class Cache{
	private List<MessagingInterface> serviceList = new List<MessagingInterface>()
	
	public void addService(MessagingInterface ms){
		List.add(ms)
	} 
	
	public MessagingInterface getService(String ms){
		return List.get(ms)
	}
}
```

и чтобы передавать к кэш сервйся нам нужно создать инициализирующий класс

```java
public class InitialContext{
	public MessagingInterface loadService(String service_name){
		if(service_name.equalIgnoreCase("makesms")){
			return new MakeSMS()
		}
		if(service_name.equalIgnoreCase("makeemail")){
			return new MakeEmail()
		}
	}
}
```

и наконец создадим сам Сервайс локатор

```java

public class ServiceLocator{
	Cache cache = new Cache()
	
	public MesagingInterface getClass(String obj){
        MessagingService service = cache.getService(serviceName);

        if (service != null) {
            return service;
        }

        InitialContext context = new InitialContext();
        
        MessagingService service1 = (MessagingService) context
          .lookup(serviceName);
          
        cache.addService(service1);
        
        return service1;
	}
}

```

И вот само использвоание:

```java
ServiceLocator sl = new ServiceLocator()

MakeSMS makeSms = sl.getClass("makeSMS")
// ... 

```

Как видно это довольно примитивный код, тут нет центрального Composition Root, хотя, есть InitialContext, но в нем не создается cache и сам Service locator

Этот паттерн используется довольно редко и бывает оправдан разе что в небольших проектах

parent::[[IoC]]