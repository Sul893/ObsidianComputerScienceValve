---
tags: [spring, security]
---

# SecurityContext

**`SecurityContext`** — контейнер данных о текущей аутентификации пользователя. Хранит объект [[Principal]] (Authentication).

## Контракт

```java
public interface SecurityContext extends Serializable {
    Authentication getAuthentication();
    void setAuthentication(Authentication authentication);
}
```

В стандартной имплементации `SecurityContextImpl`:

```java
public class SecurityContextImpl implements SecurityContext {
    private Authentication authentication;
    // getAuthentication()/setAuthentication()
}
```

## Содержимое Principal

Объект [[Principal]] (Authentication), который хранится в контексте, содержит:

- **`principal`** — идентификатор пользователя (обычно `username` или `UserDetails`)
- **`credentials`** — пароль (как правило, **очищается** сразу после успешной аутентификации)
- **`authorities`** — роли и права (granted authorities)
- **`authenticated`** — булево: успешно пройдена аутентификация или нет (false если аноним)
- **`details`** — дополнительные детали (IP, sessionId)

## Жизненный цикл

Создаётся [[AuthProvider]] при успешной аутентификации. Сохраняется в [[SecurityContextHolder]] и живёт в рамках одного HTTP-запроса (по умолчанию strategy `MODE_THREADLOCAL`). Между запросами контекст восстанавливается из репозитория (например, из HTTP-сессии) фильтром `SecurityContextHolderFilter`.

## Область видимости

`SecurityContext` доступен в любом месте обработки запроса: в [[Security Filter]], в контроллерах (`@AuthenticationPrincipal`), в сервисах — через `SecurityContextHolder.getContext().getAuthentication()`.

## Полный поток

```
HTTP-запрос
   ↓
SecurityContextHolderFilter (load context)
   ↓ из репозитория (HttpSession) ← [SecurityContext stored]
SecurityContextHolder.setContext(loaded)
   ↓
UsernamePasswordAuthenticationFilter
   ↓ создаёт token, отдаёт AuthManager → AuthProvider
   ↓ при успехе: SecurityContextHolder.getContext().setAuthentication(authenticatedToken)
   ↓
... контроллеры запускаются → @AuthenticationPrincipal == SecurityContext.getAuthentication().getPrincipal()
   ↓
AuthorizationFilter (authorize) проверяет authorities из контекста
   ↓
SecurityContextHolderFilter (save context)
   ↓ сохранить в репозиторий (HttpSession → JSESSIONID)
   ↓ очистить ThreadLocal
   ↓
HTTP-ответ
```

## Доступ к Principal из контроллера

```java
@GetMapping("/me")
public ResponseEntity<?> me(@AuthenticationPrincipal UserDetails user) {
    if (user == null) return ResponseEntity.status(401).build();
    return ResponseEntity.ok(Map.of("name", user.getUsername(),
                                    "roles", user.getAuthorities()));
}
```

## Хранение контекста между запросами

- **HttpSession** (по умолчанию через `HttpSessionSecurityContextRepository`): контекст привязан к JSESSIONID.
- **Stateless (JWT)**: контекст рекреируется при каждом запросе из JWT в фильтре, в сессии не сохраняется. `SecurityContext` после запроса просто выбрасывается.

parent:: [[SecurityContextHolder]]
friend:: [[Principal]]
friend:: [[AuthProvider]]