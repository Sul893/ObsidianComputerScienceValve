---
tags: [spring, security]
---

# AuthManager

**`AuthenticationManager` (AuthManager)** — главный интерфейс аутентификации в Spring Security. Точка входа для проверки учётных данных пользователя.

## Роль

Spring Security явно разделяет понятия:
- **Аутентификация (Authentication)** — "кто ты?" — за это отвечает `AuthManager`
- **Авторизация (Authorization)** — "что тебе можно?" — за это отвечает `AuthorizationManager`/`AuthorizationFilter`

`AuthManager` принимает запрос на аутентификацию (объект `Authentication` с `principal` и `credentials`) от [[Security Filter]] (через [[additionalFilters()]]), делегирует проверку одному или нескольким [[AuthProvider]] и возвращает заполненный [[Principal]] при успехе, либо выбрасывает `AuthenticationException` при неудаче.

## Контракт

```java
public interface AuthenticationManager {
    Authentication authenticate(Authentication authentication)
            throws AuthenticationException;
}
```

Возвращаемый объект `Authentication` (= Principal) содержит principal, authorities и флаг `authenticated=true`. Контейнер кладёт его в [[SecurityContext]] (через [[SecurityContextHolder]]).

## Реализация по умолчанию

Реализация по умолчанию — [[ProviderManager]], который перебирает список зарегистрированных [[AuthProvider]] и возвращает результат первого, поддержавшего тип аутентификации.

## Полный пример: use в фильтре

```java
public class JwtAuthenticationFilter extends OncePerRequestFilter {
    private final AuthenticationManager authManager;

    public JwtAuthenticationFilter(AuthenticationManager am) {
        this.authManager = am;
    }

    @Override
    protected void doFilterInternal(HttpServletRequest req,
                                     HttpServletResponse resp,
                                     FilterChain chain) throws IOException, ServletException {
        String token = parseToken(req);
        if (token != null) {
            var unauth = new JwtAuthenticationToken(token); // неаутентифицированный токен
            try {
                Authentication auth = authManager.authenticate(unauth);
                SecurityContextHolder.getContext().setAuthentication(auth);
            } catch (AuthenticationException ex) {
                SecurityContextHolder.clearContext();
            }
        }
        chain.doFilter(req, resp);
    }
}
```

## Получение AuthenticationManager в SecurityConfig

```java
@Configuration
public class SecurityConfig {
    private final UserDetailsService uds;
    private final PasswordEncoder encoder;

    public SecurityConfig(UserDetailsService uds, PasswordEncoder enc) {
        this.uds = uds;  this.encoder = enc;
    }

    @Bean
    public AuthenticationManager authManager(HttpSecurity http) throws Exception {
        AuthenticationManagerBuilder builder = http.getSharedObject(AuthenticationManagerBuilder.class);
        builder.userDetailsService(uds).passwordEncoder(encoder);
        return builder.build();  // возвращает ProviderManager
    }
}
```

## Схема

```
Security Filter (Username Password, JWT, ...)
        │ создаёт неаутентифицированный Authentication
        ↓
AuthManager.authenticate(auth)
        │
   ProviderManager (по умолчанию)
        │     ↓ сравни supports() для каждого AuthProvider
    ┌───┴───┬────────────────┐
 DAOAuthProvider  OAuth2Provider  JwtAuthProvider
 (БД, пароль)     (Google, GitHub) (секретный ключ)
        │
        ↓ успех
 возвращается Principal (authenticated=true)
настраивается SecurityContext.setAuthentication(auth) в фильтре
```

friend:: [[additionalFilters()]]
friend:: [[AuthProvider]]
friend:: [[Principal]]
friend:: [[ProviderManager]]
friend:: [[SecurityContext]]
child:: [[ProviderManager]]
parent:: [[SecurityConfig]]