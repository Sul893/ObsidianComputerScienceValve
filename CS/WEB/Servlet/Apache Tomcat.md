---
tags: [web, servlet]
---

# Apache Tomcat

**Apache Tomcat** — [[ServletContainer]] (контейнер сервлетов) и web-контейнер. По сути — веб-часть сервера, которая принимает, перенаправляет и отправляет HTTP-запросы.

## Как это работает

1. При инициализации Tomcat создаёт слушающий сокет, используя [[Socket API]].
2. При поступлении соединения Tomcat читает байты из сокета и формирует объект HTTP-запроса (`HttpServletRequest`).
3. Tomcat вызывает нужный [[Servlet]], передавая ему запрос и объект ответа.
4. После того как сервлет заполнил ответ (`HttpServletResponse`), Tomcat сериализует его в байты и отправляет обратно через сокет.

## Архитектура Tomcat

```
Connector (HTTP/NIO2)
   ├─ ProtocolHandler  → EndPoint (NioEndpoint, workerPool)
   └─ Processor        → Http11Processor.parseRequest() → Coyote Request
Engine (Catalina)
   ├─ Host (localhost)
   │     └─ Context (/myapp)              — одно приложение
   │           ├─ FilterChain + Filters
   │           └─ Wrapper
   │                  └─ ServletInstance.service(req,res)
ApplicationMapper    (path → Context → Wrapper)
```

- **Connector** — сетевой слой (привязка к порту; поток worker принимает запросы и парсит HTTP)
- **Coyote** — протокольный уровень Tomcat (HTTP/AJP)
- **Catalina** — сервлет-контейнер. Engine → Host → Context → Wrapper
- **Mapper** — выбирает, какому веб-приложению и сервлету соответствует URL

## Предоставляет

- [[ServletContext]] — общее окружение для всех сервлетов приложения
- Управление жизненным циклом сервлетов (`init`/`service`/`destroy`)
- Реализацию [[Socket]]-уровня для приёма HTTP
- Пул потоков для обработки запросов (thread-per-request)
- JSP-движок (Jasper)
- Поддержка HTTPS, WebSockets, асинхронных запросов (`AsyncContext`)

## Конфигурация

Tomcat настраивается через `server.xml`:

```xml
<Connector port="8080" protocol="HTTP/1.1"
           maxThreads="200" minSpareThreads="10"
           acceptCount="100" connectionTimeout="20000"/>

<Host name="localhost" appBase="webapps">
    <Context path="/myapp" docBase="/var/webapps/myapp"/>
</Host>
```

![[Pasted image 20260609121341.png]]

friend:: [[Socket API]]
friend:: [[Socket]]
friend:: [[ServletContext]]
friend:: [[Servlet]]
parent:: [[ServletContainer]]