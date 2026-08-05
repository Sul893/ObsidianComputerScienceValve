---
tags: [spring, security]
---

# PasswordEncoder

**`PasswordEncoder`** — интерфейс Spring Security для хеширования паролей. Используется [[AuthProvider]] (в частности [[DAOAuthProvider]]) для проверки хеша пароля пользователя при аутентификации.

## Роль

Пароли в хранилище (БД) никогда не хранятся в открытом виде — только хешированными через соль (salt). При аутентификации `DAOAuthProvider` получает хеш `UserDetails.password`, передаёт его и введённый пароль в `PasswordEncoder.matches(rawPassword, hash)` для сравнения.

Хорошая реализация использует **адаптивные key-stretching функции** (bcrypt, Argon2, PBKDF2) — намеренно медленные, чтобы перебор (brute-force) был невыгоден. Соль предотвращает радужные таблицы.

## Контракт

```java
public interface PasswordEncoder {
    String encode(CharSequence rawPassword);
    boolean matches(CharSequence rawPassword, String encodedPassword);
    boolean upgradeEncoding(String encodedPassword); // true → рекомендация пере-хешировать при большей силе
}
```

## Основные реализации

| Реализация | Алгоритм | Примечание |
|------------|----------|------------|
| `BCryptPasswordEncoder` | bcrypt | **Рекомендуемый**; salt встроена, медленный → устойчивость к перебору |
| `Pbkdf2PasswordEncoder` | PBKDF2 | Стандарт NIST |
| `SCryptPasswordEncoder` | scrypt | Устойчив к ASIC-атакам |
| `Argon2PasswordEncoder` | Argon2 | Современный, рекомендация OWASP |
| `NoOpPasswordEncoder` | без шифрования | **Только для тестов/демо**; в прод не использовать |

## Конфигурация

Обычно регистрируется как бин в [[SecurityConfig]]:

```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder(); // default strength=10
}
```

## Использование при регистрации

```java
@Service
public class UserService {
    private final UserRepository repo;
    private final PasswordEncoder encoder;

    public UserService(UserRepository r, PasswordEncoder e) {
        this.repo = r; this.encoder = e;
    }

    public void register(String username, String rawPw) {
        UserEntity u = new UserEntity();
        u.setUsername(username);
        u.setPasswordHash(encoder.encode(rawPw)); // соль генерируется внутри encode()
        repo.save(u);
    }

    public boolean checkPassword(UserEntity u, String rawPw) {
        return encoder.matches(rawPw, u.getPasswordHash());
    }
}
```

## Полный пример bcrypt hash

```java
PasswordEncoder enc = new BCryptPasswordEncoder(12); // strength 2^12 = 4096 rounds

String raw = "s3cret";
String h1 = enc.encode(raw); // -> "$2a$12$randomsalt..."
String h2 = enc.encode(raw); // -> "$2a$12$otherSalt..."  (разная соль → разные хеши)

enc.matches(raw, h1);   // true
enc.matches(raw, "bad"); // false
enc.upgradeEncoding(h1); // true если нужно усилить (например, если сила tăng)
```

## Динамическая смена encoder (DelegatingPasswordEncoder)

Если есть старые пароли с MD5/SHA, переходят через `DelegatingPasswordEncoder`, который хранит префикс алгоритма в хешe: `{bcrypt}$2a$12$...`, `{sha256}...`. При login можно прозрачно пере-хешировать в новый алгоритм:

```java
String idForEncode = "bcrypt";
Map<String, PasswordEncoder> encoders = new HashMap<>();
encoders.put("bcrypt", new BCryptPasswordEncoder());
encoders.put("sha256", new StandardPasswordEncoder());  // legacy
encoders.put("noop", NoOpPasswordEncoder.getInstance());

PasswordEncoder encoder = new DelegatingPasswordEncoder(idForEncode, encoders);
// Хранение: "{bcrypt}$2a$..." → matches автоматически определяет алгоритм
```

friend:: [[AuthProvider]]
friend:: [[DAOAuthProvider]]
friend:: [[UserDetailsService]]
friend:: [[SecurityConfig]]