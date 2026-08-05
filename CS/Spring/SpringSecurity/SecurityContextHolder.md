---
tags: [spring, security]
---

# SecurityContextHolder

**`SecurityContextHolder`** — абстракция, через которую Spring получает доступ к текущему [[SecurityContext]] (и, как следствие, к текущему [[Principal]]). Базовый класс со статическим API.

## Зачем нужен

Потоку обработки HTTP-запроса нужно "помнить", кто делает запрос. `SecurityContextHolder` решает задачу: один поток обработки запроса → один текущий SecurityContext. Когда любой компонент Spring Security (или вообще прикладной код) хочет узнать текущего пользователя, он обращается к `SecurityContextHolder.getContext().getAuthentication()`.

## Стратегии хранения

Через паттерн Strategy `SecurityContextHolder` выбирает, как хранить контекст:

- **`MODE_THREADLOCAL`** (по умолчанию) — `SecurityContext` привязан к текущему потоку через `ThreadLocal`. Идеально для thread-per-request.
- **`MODE_INHERITABLETHREADLOCAL`** — `SecurityContext` наследуется дочерними потоками (например, при создании `@Async` задач из запроса). Внимание: при использовании пула потоков нужно явно очищать.
- **`MODE_GLOBAL`** — статическое глобальное хранение, для не-веб приложений.

Стратегия задаётся через `SecurityContextHolder.setStrategyName(...)`:

```java
SecurityContextHolder.setStrategyName(SecurityContextHolder.MODE_INHERITABLETHREADLOCAL);
```

## Жизненный цикл за запрос

1. В начале запроса фильтр `SecurityContextHolderFilter` загружает [[SecurityContext]] из `RequestAttributeSecurityContextRepository` и устанавливает в `SecurityContextHolder`
2. Фильтр аутентификации (например `UsernamePasswordAuthenticationFilter`) при успехе создаёт [[Principal]] и кладёт в `SecurityContextHolder.getContext().setAuthentication(...)`
3. Прикладной код и фильтры авторизации обращаются к `SecurityContextHolder` для получения текущего пользователя
4. В конце запроса фильтр очищает `ThreadLocal`

## Пример использования в коде

```java
// Доступ к текущему Principal — любой слой приложения
Authentication auth = SecurityContextHolder.getContext().getAuthentication();
if (auth != null && auth.isAuthenticated()) {
    Object raw = auth.getPrincipal();
    String username;
    if (raw instanceof UserDetails ud) {
        username = ud.getUsername();
    } else {
        username = raw.toString();
    }
    Collection<? extends GrantedAuthority> roles = auth.getAuthorities();
}
```

## Удобная альтернатива — @AuthenticationPrincipal

В контроллерах лучше использовать параметр `@AuthenticationPrincipal` (берётся из того же источника, без ручного доступа к holder):

```java
@GetMapping("/me")
public Map<String, String> me(@AuthenticationPrincipal UserDetails user) {
    return Map.of("username", user.getUsername());
}
```

## Async / Reactive

Для `@Async` задач — `MODE_INHERITABLETHREADLOCAL` или явная передача через `DelegatingSecurityContextRunnable`:

```java
Runnable task = new DelegatingSecurityContextRunnable(myTask);
taskExecutor.execute(task);
```

В Spring WebFlux вместо `SecurityContextHolder` используется `ReactiveSecurityContextHolder` (контекст реактивный, не thread-local):

```java
Mono.fromSupplier(...)
    .contextWrite(ReactiveSecurityContextHolder.withAuthentication(auth));
```

## Важно

Прямое использование `SecurityContextHolder.getContext().getAuthentication()` в бизнес-коде — прозрачный путь. В контроллерах удобнее параметр `@AuthenticationPrincipal`, который берётся из того же источника.

friend:: [[SecurityContext]]
friend:: [[Principal]]
parent:: [[FilterChainProxy]]