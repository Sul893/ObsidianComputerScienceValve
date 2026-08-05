---
tags: [spring, security]
---

# SecurityFilterChain

**`SecurityFilterChain`** — цепочка [[Security Filter]], через которую проходит каждый HTTP-запрос. Описывается в [[SecurityConfig]] через `HttpSecurity` DSL.

## Контракт

```java
public interface SecurityFilterChain {
    boolean matches(HttpServletRequest request);   // подходит ли этот запрос
    List<Filter> getFilters();                     // фильтры, которые нужно выполнить
}
```

## Как работает

Spring Security создаёт один [[FilterChainProxy]] с несколькими `SecurityFilterChain`. Каждая срабатывает на определённые URL-паттерны запроса (сопоставление по `requestMatcher`). Для запроса берётся **первая** подходящая цепочка, остальные игнорируются.

Это позволяет:
- Отдельные правила для REST-эндпойнтов (`/api/**` — JWT) и веб-страниц (`/**` — form-login)
- Отключить безопасность для публичных ресурсов (`/public/**`)

## Стандартные фильтры в цепочке (порядок важен)

- `DisableEncodeUrlFilter`
- `ForceEagerSessionCreationFilter`
- `ChannelProcessingFilter` (https)
- `WebAsyncManagerIntegrationFilter`
- `SecurityContextHolderFilter` — загружает [[SecurityContext]]
- `HeaderWriterFilter`
- `CorsFilter`
- `CsrfFilter`
- `LogoutFilter`
- `UsernamePasswordAuthenticationFilter` — обрабатывает форму логина
- `BasicAuthenticationFilter` — HTTP Basic Auth
- `RequestCacheAwareFilter`
- `SecurityContextHolderAwareRequestFilter`
- `AnonymousAuthenticationFilter`
- `ExceptionTranslationFilter` — перехватывает ошибки авторизации
- `AuthorizationFilter` (бывш. `FilterSecurityInterceptor`) — проверяет права доступа

Полный список см. в [[Security Filter]].

## Пример: конфигурация одной цепочки

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http
        .securityMatcher("/**")  // матчит всё (явно не обязательно)
        .authorizeHttpRequests(auth -> auth
            .requestMatchers("/login", "/css/**").permitAll()
            .requestMatchers("/admin/**").hasRole("ADMIN")
            .anyRequest().authenticated())
        .formLogin(form -> form.loginPage("/login"))
        .csrf(Customizer.withDefaults());
    return http.build();
}
```

## Пример: несколько цепочек

```java
@Bean @Order(1)
SecurityFilterChain apiChain(HttpSecurity http) throws Exception {
    http
        .securityMatcher("/api/**")
        .csrf(c -> c.disable())
        .sessionManagement(s -> s.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
        .authorizeHttpRequests(a -> a.anyRequest().authenticated())
        .oauth2ResourceServer(o -> o.jwt(Customizer.withDefaults()));
    return http.build();
}

@Bean @Order(2)
SecurityFilterChain webChain(HttpSecurity http) throws Exception {
    http
        .authorizeHttpRequests(a -> a.anyRequest().authenticated())
        .formLogin(Customizer.withDefaults());
    return http.build();
}
```

## Просмотр зарегистрированных фильтров (отладка)

```java
http.csrf(c -> c.disable()).headers(h -> h.frameOptions(f -> f.disable()));
// Можно актуатором /actuator/loggers + выводить цепочку вручную
```

parent:: [[FilterChainProxy]]
friend:: [[Security Filter]]
friend:: [[SecurityConfig]]