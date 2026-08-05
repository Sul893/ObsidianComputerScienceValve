---
tags: [spring, security]
---

# ProviderManager

**`ProviderManager`** — реализация [[AuthManager]] по умолчанию в Spring Security. Содержит список [[AuthProvider]] и перебирает их при аутентификации.

## Контракт (упрощённый исходный код)

```java
public class ProviderManager implements AuthenticationManager {
    private final List<AuthenticationProvider> providers;
    private AuthenticationManager parent; // optional

    @Override
    public Authentication authenticate(Authentication auth) throws AuthenticationException {
        Class<? extends Authentication> authType = auth.getClass();
        AuthenticationException lastException = null;

        for (AuthenticationProvider provider : providers) {
            if (!provider.supports(authType)) continue;
            try {
                Authentication result = provider.authenticate(auth);
                if (result != null) {
                    copyDetails(auth, result);
                    return result; // успех — немедленный возврат
                }
            } catch (AuthenticationException ex) {
                lastException = ex;
                // continue на следующий провайдер
            }
        }

        // Ни один локальный не справился — пробуем родителя
        if (parent != null) return parent.authenticate(auth);

        if (lastException != null) throw lastException;
        throw new ProviderNotFoundException("No provider for " + authType.getName());
    }
}
```

## Как работает

1. Вызывается `authenticate(Authentication)` с объектом аутентификации от фильтра
2. Для каждого [[AuthProvider]] проверяется `supports(Class<?>)` — подходит ли он под тип аутентификации
3. Первый подходящий провайдер пытается аутентифицировать:
   - **Успех** — возвращается заполненный [[Principal]], остальные провайдеры не вызываются
   - `AuthenticationException` от одного провайдера → `ProviderManager` переходит к следующему
   - Ни один не справился → выбрасывается `ProviderNotFoundException`

## Parent Manager

`ProviderManager` может иметь ссылку на родительский `AuthenticationManager` (необязательно): если ни один провайдер локального списка не поддерживает тип, запрос делегируется родителю. Это формирует иерархию менеджеров.

```java
// composite провайдеры
AuthenticationManager parent =
    new ProviderManager(List.of(jwtProvider, basicProvider));
AuthenticationManager child  =
    new ProviderManager(List.of(daoProvider), parent);
```

## Конфигурация (через SecurityConfig)

Обычно создаётся в [[SecurityConfig]], где через `AuthenticationManagerBuilder` указывается список провайдеров:

```java
@Bean
public AuthenticationManager authManager(HttpSecurity http) throws Exception {
    return http.getSharedObject(AuthenticationManagerBuilder.class)
               .authenticationProvider(daoAuthProvider)
               .authenticationProvider(oauth2Provider)
               .authenticationProvider(jwtProvider)
               .build(); // возвращает ProviderManager с этими провайдерами
}
```

parent:: [[AuthManager]]
friend:: [[AuthProvider]]
friend:: [[SecurityConfig]]