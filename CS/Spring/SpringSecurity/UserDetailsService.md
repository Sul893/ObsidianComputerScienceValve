---
tags: [spring, security]
---

# UserDetailsService

**`UserDetailsService`** — интерфейс Spring Security для загрузки данных пользователя из хранилища (БД, LDAP, In-Memory и т.д.) по имени. Используется [[DAOAuthProvider]] для получения учётных данных в процессе form-login аутентификации.

## Контракт

Единственный метод:

```java
public interface UserDetailsService {
    UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
}
```

Возвращает объект `UserDetails` (пользовательское DTO с username, password (хэш), authorities, флагами аккаунта). Если пользователь не найден — выбрасывается `UsernameNotFoundException`.

## UserDetails контракт

```java
public interface UserDetails extends Serializable {
    String getUsername();
    String getPassword();               // хеш
    Collection<? extends GrantedAuthority> getAuthorities();
    boolean isAccountNonExpired();
    boolean isAccountNonLocked();
    boolean isCredentialsNonExpired();
    boolean isEnabled();
}
```

## Реализации

- **`JdbcUserDetailsManager`** — чтение пользователей и ролей из БД через JDBC
- **`InMemoryUserDetailsManager`** — хранение в памяти (для демо и тестов)
- **Своя реализация** — часто пишут в проекте, обёртывая вызов репозитория `UserRepository`, JPA или LDAP

## In-Memory example

```java
@Bean
public UserDetailsService users(PasswordEncoder encoder) {
    UserDetails admin = User.builder()
        .username("admin")
        .password(encoder.encode("secret"))
        .roles("ADMIN")
        .build();
    UserDetails user = User.builder()
        .username("alice")
        .password(encoder.encode("pass"))
        .roles("USER")
        .build();
    return new InMemoryUserDetailsManager(admin, user);
}
```

## JDBC example

```java
@Bean
public UserDetailsService jdbcUsers(DataSource dataSource) {
    JdbcUserDetailsManager mgr = new JdbcUserDetailsManager(dataSource);
    // Использует default SQL, можно переопределить через mgr.setUsersByUsernameQuery(...)
    return mgr;
}
```

## Связь с проверкой пароля

`DAOAuthProvider` использует UserDetailsService только для загрузки `UserDetails`. Проверка пароля (скрытого в `UserDetails.password`) выполняется отдельно через [[PasswordEncoder]].

## Пример кастомной реализации (JPA + Lombok)

```java
@Service
@RequiredArgsConstructor
public class JpaUserDetailsService implements UserDetailsService {
    private final UserRepository userRepository;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        UserEntity u = userRepository.findByUsername(username)
            .orElseThrow(() -> new UsernameNotFoundException("Not found: " + username));
        return User.withUsername(u.getUsername())
                   .password(u.getPasswordHash()) // уже bcrypt хэш
                   .authorities(u.getRoles().stream()
                       .map(r -> new SimpleGrantedAuthority("ROLE_" + r))
                       .toList())
                   .accountExpired(!u.isActive())
                   .accountLocked(u.isLocked())
                   .credentialsExpired(false)
                   .disabled(!u.isEnabled())
                   .build();
    }
}
```

```java
// Repository
public interface UserRepository extends JpaRepository<UserEntity, Long> {
    Optional<UserEntity> findByUsername(String username);
}
```

friend:: [[DAOAuthProvider]]
friend:: [[AuthProvider]]
friend:: [[PasswordEncoder]]