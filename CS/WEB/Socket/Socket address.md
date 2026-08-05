---
tags: [web, socket]
---

# Socket Address

**Socket Address** — комбинация Protocol + IP + Port, идентифицирующая конкретный [[Socket]] в сети. Тройка однозначно определяет точку подключения между двумя узлами.

Полный адрес соединения TCP — это **пара** socket addresses: локальный и удалённый. Например, соединение `(192.168.1.1:80, 10.0.0.5:51324)`.

## Состав

- **Protocol** — способ передачи данных:
  - `TCP` — гарантированная доставка, упорядоченность, connection-oriented (используется HTTP, FTP, SMTP)
  - `UDP` — быстрая доставка без гарантий, без установки соединения (используется DNS, DHCP, стриминг, VoIP)
- **IP** — идентификатор хоста в сети: `IPv4` (например, `192.168.1.1`, 32 бита) или `IPv6` (например, `2001:db8::1`, 128 бит)
- **Port** — идентификатор конкретного приложения/процесса на хосте, диапазон `0–65535` (16 бит). Диапазоны:
  - `0–1023` — well-known ports (80 HTTP, 443 HTTPS, 22 SSH, 53 DNS)
  - `1024–49151` — registered ports
  - `49152–65535` — dynamic/ephemeral (временные порты клиентов)

## Формат

`tcp://192.168.1.1:8080` — протокол TCP, хост 192.168.1.1, порт 8080.

## Пример на Java (привязка адреса)

```java
// Серверный сокет привязывается к адресу
InetSocketAddress addr = new InetSocketAddress("0.0.0.0", 8080);
ServerSocket server = new ServerSocket();
server.bind(addr);                 // Protocol=TCP, IP=0.0.0.0, Port=8080

// Клиентский сокет подключается к адресу
Socket client = new Socket();
client.connect(new InetSocketAddress("93.184.216.34", 80));
// теперь локальный адрес:порт клиента — ephemeral (назначен ОС)
```

Узнать адреса:

```java
Socket s = new Socket("example.com", 80);
System.out.println(s.getLocalSocketAddress());  // /192.168.1.5:51324
System.out.println(s.getRemoteSocketAddress()); // /93.184.216.34:80
```

parent:: [[Socket]]