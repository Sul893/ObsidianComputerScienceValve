---
tags: [spring, security]
---

# Security Filter

**Security Filter** — отдельный фильтр в цепочке [[SecurityFilterChain]]. Каждый фильтр отвечает за определённый аспект безопасности: аутентификацию, авторизацию, защиту от CSRF/CORS, обработку logout и т.д.

Все HTTP-запросы проходят через эти фильтры последовательно, до попадания в [[DispatcherServlet]]. Порядок фильтров в цепочке строго задан Spring Security.

## Контракт

Каждый фильтр реализует стандартный интерфейс `jakarta.servlet.Filter`:

```java
public interface Filter {
    void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain)
        throws IOException, ServletException;
}
```

## Основные фильтры

- **`SecurityContextHolderFilter`** — загружает [[SecurityContext]] из репозитория в начале запроса, сохраняет в конце. Связан с [[SecurityContextHolder]].
- **`UsernamePasswordAuthenticationFilter`** — обрабатывает POST-форму логина. Извлекает username/password, передаёт в [[AuthManager]]. При успехе создаёт [[Principal]] и сохраняет в [[SecurityContext]].
- **`BasicAuthenticationFilter`** — обрабатывает HTTP Basic Auth (заголовок `Authorization: Basic ...`).
- **`OAuth2AuthorizationRequestRedirectFilter`** — обрабатывает OAuth2 login, редиректит на внешнего провайдера ([[OAuth2Provider]]).
- **`BearerTokenAuthenticationFilter`** — обрабатывает Bearer-токен (JWT).
- **`AnonymousAuthenticationFilter`** — если пользователь не аутентифицирован, создаёт анонимного Principal (для разрешённых public URL).
- **`ExceptionTranslationFilter`** — переводит исключения `AuthenticationException` / `AccessDeniedException` в соответствующие HTTP-ответы (401/403 или редирект на страницу логина).
- **`AuthorizationFilter`** (бывш. `FilterSecurityInterceptor`) — финальный фильтр, проверяет права доступа (authorities) к URL по паттерну.
- **`LogoutFilter`** — обрабатывает запрос logout, инвалидирует сессию и очищает SecurityContextHolder.
- **`CsrfFilter`** — проверяет CSRF-токен для POST/PUT/DELETE.
- **`CorsFilter`** — добавляет CORS-заголовки.
- **`HeaderWriterFilter`** — добавляет security-заголовки (`X-Content-Type-Options`, `X-Frame-Options`).

## Пример: собственный фильтр аутентификации (JWT)

```java
public class JwtAuthenticationFilter extends OncePerRequestFilter {

    private final JwtService jwtService;

    @Override
    protected void doFilterInternal(HttpServletRequest req,
                                    HttpServletResponse resp,
                                    FilterChain chain) throws ServletException, IOException {
        String header = req.getHeader("Authorization");
        if (header == null || !header.startsWith("Bearer ")) {
            chain.doFilter(req, resp); // продолжить, AnonymousAuthenticationFilter создаст Anonymous
            return;
        }

        String token = header.substring(7);
        if (jwtService.isValid(token)) {
            UserDetails user = jwtService.parse(token);
            var auth = new UsernamePasswordAuthenticationToken(
                user, null, user.getAuthorities());   // новый Principal
            SecurityContextHolder.getContext().setAuthentication(auth);
        }
        chain.doFilter(req, resp);
    }
}
```

Регистрация через дополнительный метод [[additionalFilters()]] в [[SecurityConfig]]:

```java
http.addFilterBefore(jwtAuthenticationFilter,
                     UsernamePasswordAuthenticationFilter.class);
```

## Пример: простой фильтр заголовков

```java
public class AddServerHeaderFilter implements Filter {
    public void doFilter(ServletRequest rq, ServletResponse rs, FilterChain c)
            throws IOException, ServletException {
        HttpServletResponse resp = (HttpServletResponse) rs;
        resp.setHeader("X-Server", "MyApp/2.1");
        c.doFilter(rq, rs);
    }
}
```

## Аутентификация через AuthManager

Фильтры аутентификации делегируют проверку в [[AuthManager]], который конфигурируется (через [[additionalFilters()]]) в [[SecurityConfig]].

## Просмотр зарегистрированных фильтров

```java
// Через логирование SecurityBuilder
//org.springframework.security.web.FilterChainProxy  TRACE
// → в log завидим цепочку фильтров в порядке выполнения
```

parent:: [[SecurityFilterChain]]
friend:: [[AuthManager]]
friend:: [[SecurityContextHolder]]
friend:: [[SecurityContext]]