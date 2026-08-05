---
tags: [spring, security]
---

# AuthProvider

**`AuthenticationProvider` (AuthProvider)** — интерфейс, выполняющий фактическую проверку учётных данных. Принимает запрос на аутентификацию от [[AuthManager]] (в типичной конфигурации — от [[ProviderManager]], перебирающего провайдеров).

## Роль

Каждый AuthProvider поддерживает определённый тип аутентификации (через метод `supports(Class<?>)`). Провайдер:
1. Извлекает principal/credentials из переданного `Authentication`
2. Проверяет их (например, сравнивает хеш пароля с [[PasswordEncoder]])
3. При успехе возвращает заполненный [[Principal]] с authorities и `authenticated=true`
4. При неудаче выбрасывает `BadCredentialsException`, `AuthenticationException`

## Контракт

```java
public interface AuthenticationProvider {
    Authentication authenticate(Authentication authentication)
        throws AuthenticationException;

    boolean supports(Class<?> authentication);
}
```

## Полный пример: кастомный провайдер

```java
public class ApiKeyAuthenticationProvider implements AuthenticationProvider {

    private final Set<String> validKeys;

    public ApiKeyAuthenticationProvider(Set<String> keys) { this.validKeys = keys; }

    @Override
    public Authentication authenticate(Authentication auth) throws AuthenticationException {
        String apiKey = (String) auth.getPrincipal();

        if (!validKeys.contains(apiKey)) {
            throw new BadCredentialsException("Invalid API key");
        }
        return new ApiKeyAuthenticationToken(apiKey, Collections.emptyList());
    }

    @Override
    public boolean supports(Class<?> authType) {
        return ApiKeyAuthenticationToken.class.isAssignableFrom(authType);
    }
}
```

```java
@Configuration
public class SecurityConfig {
    @Bean
    public AuthenticationProvider apiKeyProvider() {
        return new ApiKeyAuthenticationProvider(Set.of("abc-123", "xyz-789"));
    }

    @Bean
    public AuthenticationManager authManager(AuthenticationProvider apiKeyProvider) {
        return new ProviderManager(apiKeyProvider);
    }
}
```

## Основные реализации

- [[DAOAuthProvider]] (`DaoAuthenticationProvider`) — аутентификация через БД (form-login). Использует [[UserDetailsService]] и [[PasswordEncoder]].
- [[OAuth2Provider]] (`OAuth2LoginAuthenticationProvider`) — аутентификация через внешнего OAuth2-провайдера (Google, GitHub).
- `JwtAuthenticationProvider` — валидация JWT-токена.
- `RememberMeAuthenticationProvider` — поддержка "запомнить меня".

Несколько провайдеров можно зарегистрировать в одном [[ProviderManager]] — например, комбинировать DAO + OAuth2 + JWT в одном приложении.

```java
@Bean
public ProviderManager providerManager(
        AuthenticationProvider dao, AuthenticationProvider oauth, AuthenticationProvider jwt) {
    return new ProviderManager(List.of(dao, oauth, jwt));
}
```

friend:: [[AuthManager]]
friend:: [[Principal]]
friend:: [[UserDetailsService]]
friend:: [[PasswordEncoder]]
friend:: [[DAOAuthProvider]]
friend:: [[OAuth2Provider]]