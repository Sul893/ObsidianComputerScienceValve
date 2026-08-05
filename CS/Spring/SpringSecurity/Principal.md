---
tags: [spring, security]
---

# Principal

**`Principal`** (он же `Authentication Object`) — объект аутентификации в Spring Security. Интерфейс `Authentication` из Spring Security **и есть** Principal-объект; он реализует интерфейс `Principal` из `java.security`.

## Контракт

```java
public interface Authentication extends Principal, Serializable {
    Collection<? extends GrantedAuthority> getAuthorities();
    Object getCredentials();         // пароль или токен
    Object getDetails();              // доп. мета (IP, sessionId)
    Object getPrincipal();            // UserDetails / username / OAuth2User
    boolean isAuthenticated();
    void setAuthenticated(boolean isAuthenticated);
}
```

## Содержимое

Объект `Authentication` содержит:

| Поле | Описание |
|------|----------|
| `principal` | Идентификатор аутентифицированного пользователя. После form-login это `UserDetails`, при JWT — `String username`, при OAuth2 — `OAuth2User`. |
| `credentials` | Пароль / токен (обычно очищается после проверки) |
| `authorities` | Коллекция прав/ролей (`GrantedAuthority`), например `ROLE_ADMIN`, `document:read` |
| `authenticated` | `true` после успешной аутентификации; `false` для анонимного пользователя |
| `details` | Доп. мета (IP, sessionId) |

## Жизненный цикл

1. **Создание запроса** — фильтр (например, `UsernamePasswordAuthenticationFilter`) создаёт *неаутентифицированный* `UsernamePasswordAuthenticationToken` с логином/паролем
2. **Аутентификация** — [[AuthProvider]] проверяет учётные данные. При успехе создаёт *аутентифицированный* `Authentication` с заполненными `principal`, `authorities`, `authenticated=true`
3. **Хранение** — сохраняется в [[SecurityContext]], который доступен через [[SecurityContextHolder]]
4. **Использование** — во время запроса любой компонент (фильтры авторизации, контроллеры) может получить Principal из holder или через `@AuthenticationPrincipal Authentication auth`

## Специализации

- `UsernamePasswordAuthenticationToken` — имя+пароль (form-login)
- `JwtAuthenticationToken` — JWT-токен
- `OAuth2AuthenticationToken` — после успешного OAuth2-логина (`OAuth2User`)
- `AnonymousAuthenticationToken` — анонимный пользователь
- `BearerTokenAuthentication` — OAuth2 resource server (JWT)
- `TestingAuthenticationToken` — для unit-тестов

## Пример: Principal в двух состояниях

```java
// 1. Unauthenticated — создаётся в фильтре UsernamePasswordAuthenticationFilter
Authentication unauth = new UsernamePasswordAuthenticationToken(
    "alice", "secret");      // principal=username, credentials=password, authorities=golden
assert !unauth.isAuthenticated();

// 2. Authenticated — создаётся Provider'ом после проверки
Authentication auth = new UsernamePasswordAuthenticationToken(
    userDetails,            // principal=UserDetails
    null,                   // credentials=null (очищены)
    List.of(new SimpleGrantedAuthority("ROLE_USER")));
                                                    // authorities
assert auth.isAuthenticated();

// Использование в коде:
Object principal = auth.getPrincipal();
String username = (principal instanceof UserDetails ud)
    ? ud.getUsername()
    : principal.toString();
Collection<? extends GrantedAuthority> roles = auth.getAuthorities();
```

## Доступ из контроллера

```java
@GetMapping("/me")
public Map<String, Object> me(@AuthenticationPrincipal UserDetails user) {
    return Map.of(
        "username", user.getUsername(),
        "roles", user.getAuthorities()
            .stream().map(GrantedAuthority::getAuthority).toList());
}
```

## В WebFlux

```java
Mono.just(Map.of("user", "alice"))
    .contextWrite(ReactiveSecurityContextHolder.withAuthentication(auth));
```

parent:: [[SecurityContext]]
friend:: [[AuthProvider]]
friend:: [[SecurityContextHolder]]