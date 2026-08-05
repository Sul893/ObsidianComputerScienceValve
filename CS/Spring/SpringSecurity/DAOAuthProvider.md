---
tags: [spring, security]
---

# DAOAuthProvider

**`DaoAuthenticationProvider` (DAOAuthProvider)** — реализация [[AuthProvider]], выполняющая аутентификацию через данные из хранилища (БД, LDAP). Стандартный провайдер для form-login (имя пользователя + пароль из формы).

## Как работает

1. Фильтр (например, `UsernamePasswordAuthenticationFilter`) ловит POST формы логина и формирует `UsernamePasswordAuthenticationToken` с username/password (credentials в открытом виде)
2. [[AuthManager]] ([[ProviderManager]]) делегирует `DAOAuthProvider`
3. `DAOAuthProvider` через [[UserDetailsService]] загружает `UserDetails` по username. Если пользователь не найден → выбрасывает `UsernameNotFoundException`
4. Сравнивает хеш пароля credentials с хешем `UserDetails.password` через [[PasswordEncoder]]. Несовпадение → `BadCredentialsException`
5. Проверяет `isAccountNonExpired()`, `isAccountNonLocked()`, `isCredentialsNonExpired()`, `isEnabled()` — иначе своё исключение
6. При успехе возвращает [[Principal]] с `authorities` из `UserDetails`

## Контракт (упрощённый исходный код)

```java
public class DaoAuthenticationProvider extends AbstractUserDetailsAuthenticationProvider {

    private UserDetailsService userDetailsService;
    private PasswordEncoder passwordEncoder = NoOpPasswordEncoder.getInstance();

    @Override
    protected void additionalAuthenticationChecks(UserDetails user,
                                                  UsernamePasswordAuthenticationToken auth)
            throws AuthenticationException {
        String presented = (String) auth.getCredentials();
        if (!passwordEncoder.matches(presented, user.getPassword())) {
            throw new BadCredentialsException("Bad credentials");
        }
    }

    @Override
    protected UserDetails retrieveUser(String username,
                                        UsernamePasswordAuthenticationToken auth)
            throws AuthenticationException {
        try {
            return userDetailsService.loadUserByUsername(username);
        } catch (UsernameNotFoundException ex) {
            throw ex;
        }
    }
}
```

## Конфигурация

```java
@Configuration
public class SecurityConfig {

    private final UserDetailsService userDetailsService;
    private final PasswordEncoder passwordEncoder;

    public SecurityConfig(UserDetailsService uds, PasswordEncoder pe) {
        this.userDetailsService = uds; this.passwordEncoder = pe;
    }

    @Bean
    public AuthenticationProvider authenticationProvider() {
        DaoAuthenticationProvider provider = new DaoAuthenticationProvider();
        provider.setUserDetailsService(userDetailsService);
        provider.setPasswordEncoder(passwordEncoder);
        return provider;
    }

    @Bean
    public AuthenticationManager authenticationManager(
            AuthenticationProvider provider) {
        return new ProviderManager(provider);
    }
}
```

## Alternative — через builder

```java
@Bean
public AuthenticationManager authManager(HttpSecurity http) throws Exception {
    return http.getSharedObject(AuthenticationManagerBuilder.class)
               .userDetailsService(userDetailsService)
               .passwordEncoder(passwordEncoder) // возвращает DaoAuthenticationProvider
               .and().build();
}
```

## Жизненный цикл: успех / неудача

```
UsernamePasswordAuthenticationFilter
   │ создаёт unauthenticatedToken с (username, password)
   ↓
AuthManager.authenticate(token)
   ↓
DAOAuthProvider.authenticate(token)
   │
   ├→ retrieveUser(username) → UserDetailsService.loadUserByUsername()
   │   ├── UserDetails найден: { username, hash, authorities }
   │   └── не найден → UsernameNotFoundException
   ├→ additionalAuthenticationChecks:
   │   passwordEncoder.matches(presented, hash)
   │   └── несовпадение → BadCredentialsException
   ├→ checks account: isAccountNonExpired/NonLocked, isCredentialsNonExpired, isEnabled
   └→ создаёт authenticatedToken (authenticated=true, authorities)
   ↓
SecurityContextHolder.getContext().setAuthentication(authToken) в фильтре
```

parent:: [[AuthProvider]]
friend:: [[UserDetailsService]]
friend:: [[PasswordEncoder]]
friend:: [[Principal]]