---
tags: [deploy]
---

# Docker

**Docker** — платформа для контейнеризации приложений. Позволяет упаковать приложение со всеми его зависимостями (библиотеками, конфигами, runtime) в изолированный, переносимый контейнер, который одинаково работает на любой машине.

## Основные понятия

- **Image (образ)** — шаблон для создания контейнера. Содержит файловую систему с приложением и зависимостями. Описывается в `Dockerfile`.
- **Container (контейнер)** — запущенный экземпляр образа. Изолирован от хоста и других контейнеров. Легче виртуальной машины, так как разделяет ядро host-ОС.
- **Dockerfile** — текстовый файл с инструкциями по сборке образа (`FROM`, `COPY`, `RUN`, `ENV`, `CMD`, `ENTRYPOINT`).
- **Docker Compose** — инструмент для запуска multi-container приложений, описанных в `docker-compose.yml`.
- **Registry** — хранилище образов: Docker Hub, private registry (Harbor), GitHub Container Registry.
- **Volume** — механизм персистентного хранения данных, независимого от жизненного цикла контейнера.

## Пример Dockerfile для Spring Boot

```dockerfile
# build stage
FROM maven:3.9-eclipse-temurin-21 AS build
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline                    # кэш зависимостей
COPY src ./src
RUN mvn -B clean package -DskipTests

# runtime stage (легкий образ без Maven, без исходников)
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
COPY --from=build /app/target/app.jar app.jar
EXPOSE 8080
ENV JAVA_OPTS="-XX:+UseContainerSupport -XX:MaxRAMPercentage=75"
ENTRYPOINT ["sh","-c","java $JAVA_OPTS -jar app.jar"]
```

Сборка и запуск:

```bash
docker build -t myapp:1.0 .
docker run -p 8080:8080 myapp:1.0
```

## Пример docker-compose.yml (multi-container)

```yaml
version: "3.9"
services:
  app:
    build: .
    ports: ["8080:8080"]
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://db:5432/mydb
    depends_on: [db]

  db:
    image: postgres:16
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: app
      POSTGRES_PASSWORD: secret
    volumes:
      - db_data:/var/lib/postgresql/data

volumes:
  db_data:
```

Запуск: `docker compose up -d`

## Main команды

```bash
docker ps                              # список запущенных контейнеров
docker images                          # список образов
docker logs -f <container>             # логи контейнера
docker exec -it <container> sh         # войти в контейнер
docker stop <container> && docker rm <container>
docker volume ls                       # список volumes
docker network create mynet            # создать сеть
docker build -t myapp:1.0 .            # сборка
docker run --name app -p 8080:8080 -d --rm myapp:1.0
docker push registry.io/myorg/myapp:1.0
docker pull registry.io/myorg/myapp:1.0
```

## Преимущества

- Воспроизводимость окружения на всех этапах (dev → staging → prod)
- Изоляция сервисов и их зависимостей
- Быстрый запуск по сравнению с виртуальными машинами
- Удобное масштабирование с [[CI]]/[[CD]] пайплайнами и оркестраторами (Kubernetes)

## Связь с CI/CD

Используется в пайплайнах [[CI]]/[[CD]] для обеспечения одинакового окружения на всех этапах: сборка в контейнере → образ пушится в registry → на деплое образ разворачивается без изменений.

typical CI/CD flow:
```
mvn test → docker build → docker push → kubectl set image
          ↑ здесь Dockerfile определяет, что внутри образа
```

friend:: [[CI]]
friend:: [[CD]]