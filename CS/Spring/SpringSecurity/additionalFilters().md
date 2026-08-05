---
tags: [spring, security]
---

# additionalFilters()

**`additionalFilters()`** — набор методов в `HttpSecurity` DSL внутри [[SecurityConfig]], позволяющих добавить кастомные фильтры в [[SecurityFilterChain]], управляемый [[FilterChainProxy]].

## Назначение

Стандартные [[Security Filter]] охватывают типовые сценарии (form-login, Basic auth, JWT). Когда нужно расширить логику (добавить собственный JWT-фильтр, multi-tenancy, метрики), используют `addFilter()` / `addFilterBefore()` / `addFilterAfter()`.

## Расположение в цепочке

Положение фильтра в порядке критично — это влияет на доступность [[SecurityContext]] и принципала. Spring Security выставляет каждый фильтр на определённую "позицию" относительно стандартных:

```java
http.addFilterBefore(myCustomFilter, UsernamePasswordAuthenticationFilter.class);
http.addFilterAfter(myCustomFilter, BasicAuthenticationFilter.class);
http.addFilterAt(myCustomFilter, UsernamePasswordAuthenticationFilter.class); // замена
```

Кастомные фильтры, реализующие аутентификацию, в итоге передают аутентификацию в [[AuthManager]], который возвращает [[Principal]].

## API

| Метод | Что делает |
|-------|------------|
| `addFilter(Filter f)` | Вставляет фильтр в "правильную" позицию (для стандартных типов) |
| `addFilterAt(Filter f, Class<? extends Filter> anchor)` | На точную позицию anchor'а (заменяет стандартный) |
| `addFilterBefore(Filter f, Class<? extends Filter> anchor)` | Перед позицией anchor'а |
| `addFilterAfter(Filter f, Class<? extends Filter> anchor)` | После anchor'а |

## Пример: добавление JWT-фильтра

```java
@Configuration
public class SecurityConfig {

    private final JwtAuthenticationFilter jwtFilter;

    public SecurityConfig(JwtAuthenticationFilter jwtFilter) { this.jwtFilter = jwtFilter; }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(c -> c.disable())
            .sessionManagement(s -> s.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(a -> a
                .requestMatchers("/auth/**").permitAll()
                .anyRequest().authenticated())
            .addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class);
        return http.build();
    }
}
```

## Полный пример: 2 кастомных фильтра

```java
http
    .addFilterBefore(new SanitizeInputFilter(), SecurityContextHolderFilter.class)
    .addFilterAfter(new TenantContextFilter(tenantService), SecurityContextHolderFilter.class)
    .addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class);
// Цепочка: SanitizeInputFilter → SecurityContextHolderFilter → TenantContextFilter → ...
//       → UsernamePasswordAuthenticationFilter (пропущен, если JWT уже аутентифицировал)
```

## Свофильтр должен расширять OncePerRequestFilter

```java
public abstract class OncePerRequestFilter implements Filter {
    // вызывает doFilterInternal() ровно раз — даже если обёрнут forward/include
    protected abstract void doFilterInternal(HttpServletRequest req,
                                             HttpServletResponse resp,
                                             FilterChain chain) throws Exception;
}
```

parent:: [[FilterChainProxy]]
friend:: [[SecurityConfig]]
friend:: [[AuthManager]]
friend:: [[Security Filter]]