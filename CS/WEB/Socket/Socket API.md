---
tags: [web, socket]
---

# Socket API

**Socket API** — программный интерфейс [[Socket]] на 7-ом (прикладном) уровне модели OSI. Предоставляется операционной системой, которая содержит объект [[Socket#^socketKernel|Socket Kernel]].

## Основные системные вызовы

- `socket()` — создание сокета (AF_INET/AF_INET6, SOCK_STREAM/SOCK_DGRAM)
- `bind()` — привязка сокета к адресу (IP + Port)
- `listen()` — перевод сокета в режим ожидания входящих соединений (TCP-сервер; задаётся размер очереди)
- `accept()` — блокирующее принятие входящего соединения, возвращает новый сокет для клиента
- `connect()` — установка соединения (TCP-клиент)
- `send()` / `recv()` — отправка/получение данных
- `close()` — закрытие сокета и освобождение ресурсов

## Серверная сторона (TCP)

```c
int server_fd = socket(AF_INET, SOCK_STREAM, 0);          // создать TCP-сокет

struct sockaddr_in addr = {AF_INET, htons(8080), INADDR_ANY};
bind(server_fd, (struct sockaddr*)&addr, sizeof(addr));    // привязать адрес

listen(server_fd, 128);                                    // слушать

int client_fd = accept(server_fd, NULL, NULL);             // ждать клиента
char buf[4096];
int n = recv(client_fd, buf, sizeof(buf), 0);              // получить
send(client_fd, "HTTP/1.1 200 OK\r\n\r\nHi", 17, 0);        // ответить
close(client_fd);                                          // закончить
close(server_fd);
```

## Клиентская сторона (TCP)

```c
int fd = socket(AF_INET, SOCK_STREAM, 0);
struct sockaddr_in addr = {AF_INET, htons(80), inet_addr("93.184.216.34")};
connect(fd, (struct sockaddr*)&addr, sizeof(addr));        // установить соединение
send(fd, "GET / HTTP/1.1\r\nHost: x\r\n\r\n", 27, 0);
char buf[4096];
recv(fd, buf, sizeof(buf), 0);
close(fd);
```

## UDP (без установки соединения)

```c
int fd = socket(AF_INET, SOCK_DGRAM, 0);
sendto(fd, data, len, 0, &addr, addrlen);   // сразу отправляем
recvfrom(fd, buf, len, 0, &srcaddr, &addrlen);
```

## Java обёртка

Java прячет raw Socket API за классами `java.net.Socket` / `ServerSocket`:

```java
try (ServerSocket server = new ServerSocket(8080)) {
    Socket client = server.accept();
    BufferedReader r = new BufferedReader(
            new InputStreamReader(client.getInputStream()));
    String line; while ((line = r.readLine()) != null) System.out.println(line);
}
```

## Использование

 Через Socket API [[Apache Tomcat]] создаёт слушающий сокет, принимает HTTP-запросы от клиентов и отправляет HTTP-ответы. Это нижний уровень, на котором работает любой веб-сервер.

![[Pasted image 20260609112723.png]]

friend:: [[Apache Tomcat]]
friend:: [[Socket kernel]]
parent:: [[Socket]]