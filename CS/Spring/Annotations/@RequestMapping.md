---
tags: [spring, annotation, web]
---

# @RequestMapping

**`@RequestMapping`** — аннотация Spring MVC, привязывающая метод [[@Controller]] (или весь класс) к обработке HTTP-запросов по определённому пути и HTTP-методу.

## Основные атрибуты

- `value` / `path` — URL-путь (поддерживает Ant-шаблоны: `/users/{id}`)
- `method` — HTTP-метод: `GET`, `POST`, `PUT`, `DELETE`, `PATCH`
- `params` — фильтр по параметрам запроса
- `headers` — фильтр по HTTP-заголовкам
- `consumes` — фильтр по `Content-Type` запроса
- `produces` — фильтр по `Accept` клиента
- `path` поддерживает шаблоны: `/users/{id}`, `/api/v?/items`, `/files/**`

## На уровне класса и метода

```java
@RestController
@RequestMapping("/api/users")              // общий префикс для класса
public class UserRestController {

    @GetMapping("/{id}")                   // GET /api/users/{id}
    public UserDto get(@PathVariable Long id) { ... }

    @PostMapping                            // POST /api/users
    public UserDto create(@RequestBody UserCreateRequest req) { ... }

    @PutMapping("/{id}")                    // PUT /api/users/{id}
    public UserDto update(@PathVariable Long id,
                          @RequestBody UserUpdateRequest req) { ... }

    @DeleteMapping("/{id}")                 // DELETE /api/users/{id}
    public void delete(@PathVariable Long id) { ... }
}
```

## Специализации по HTTP-методу

Сокращённые аннотации с предустановленным `method`:
- `@GetMapping` — только GET
- `@PostMapping` — только POST
- `@PutMapping` — только PUT
- `@DeleteMapping` — только DELETE
- `@PatchMapping` — только PATCH

## Фильтрация по условию

### По заголовкам / параметрам

```java
@GetMapping(value = "/users", params = "active=true")
public List<User> getUsersWithStatusActive() { ... }

// Только если запрос Accept: application/json
@GetMapping(path = "/users", produces = "application/json")
public List<User> asJson() { ... }

// Только XML — другой метод
@GetMapping(path = "/users", produces = "application/xml")
public Users asXml() { ... }
```

### По Content-Type

```java
@PostMapping(path = "/users", consumes = "application/json")
public User createJson(@RequestBody User user) { ... }

@PostMapping(path = "/users", consumes = "application/xml")
public User createXml(@RequestBody User user) { ... }
```

### С шаблонами пути

| Паттерн | Что мэтчит | Пример |
|---------|-----------|--------|
| `/users/{id}` | точный id | `/users/42` |
| `/api/v?/items` | один символ | `/api/v1/items`, `/api/v2/items` |
| `/files/**` | любой под-путь | `/files/a/b/c.png` |
| `/api/{*path}` | "catch-all" | `/api/foo/bar` → path=`/foo/bar` |

## Путь запроса

Запрос проходит цепочку обработки:
[[DispatcherServlet]] → [[Handler Mapping]] → [[Handle Object]] → [[Handler Adapter]] → [[@Controller]].

Результат от контроллера рендерится через [[ViewResolver]] в [[Web page]].

ASCII-диаграмма жизненного цикла запроса:

```
[HTTP] ─→ [DispatcherServlet]
                │
                ├─ [Handler Mapping]   →  URL → Handle Object
                │
                ├─ [Handle Interceptor.preHandle]
                │
                ├─ [Handler Adapter.invoke()]  →  Controller method
                │
                ├─ [Handle Interceptor.postHandle]
                │
                ├─ [ViewResolver]      →  логическое имя → Web page
                │
                └─ [Handle Interceptor.afterCompletion]
```

friend:: [[Web page]]
friend:: [[@Controller]]
friend:: [[Handler Mapping]]
friend:: [[Handle Object]]
friend:: [[Handler Adapter]]
friend:: [[ViewResolver]]
friend:: [[DispatcherServlet]]