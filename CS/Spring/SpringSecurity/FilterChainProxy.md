---
tags: [spring, security]
---

# FilterChainProxy

**`FilterChainProxy`** — ключевой Spring-бин Spring Security, создаваемый в [[SecurityConfig]]. Управляет одной или несколькими цепочками фильтров безопасности и является единым входом во всю фильтрацию Spring Security.

Бин регистрируется под именем `springSecurityFilterChain` — именно его ищет [[DelegatingFilterProxy]].

## Роль

`DelegatingFilterProxy` (сервлетный фильтр) делегирует ему обработку каждого HTTP-запроса. `FilterChainProxy`:
- Содержит список [[SecurityFilterChain]]
- Для конкретного запроса находит первую подходящую по URL-паттерну цепочку
- Пропускает запрос через фильтры найденной цепочки ([[Security Filter]])
- Отфильтрованный запрос попадает в [[DispatcherServlet]]

## Контракт

```java
public class FilterChainProxy extends GenericFilterBean {
    private final List<SecurityFilterChain> filterChains;

    @Override
    public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain) {
        List<Filter> filters = getFilters(request);   // выбрать цепочку
        VirtualFilterChain vfc = new VirtualFilterChain(chain, filters);
        vfc.doFilter(request, response);              // пройти по фильтрам
    }

    private List<Filter> getFilters(HttpServletRequest request) {
        for (SecurityFilterChain chain : filterChains) {
            if (chain.matches(request)) return chain.getFilters();
        }
        return Collections.emptyList();
    }
}
```

## Важно

`FilterChainProxy` — это единственный фильтр, который `DelegatingFilterProxy` делегирует. Все детали (порядок фильтров, "addFilterAfter" и т.д.) конфигурируются внутри него.

После прохождения цепочки фильтров итог аутентификации помещается в [[SecurityContextHolder]] (через [[SecurityContext]] — фильтром `SecurityContextHolderFilter`).

## Несколько цепочек: как выбирается

```java
@Bean
SecurityFilterChain apiChain(HttpSecurity http)   { /* @Order(1), urlMatcher=/api/** */ }
@Bean
SecurityFilterChain webChain(HttpSecurity http)   { /* @Order(2), urlMatcher=/**       */ }
// Для запроса /api/orders → apiChain (первый матчит)
// Для запроса /dashboard → webChain (apiChain не матчит /dashboard)
```

## Пример из реестра фильтров (внутренности)

```
filterChains = [
  SecurityFilterChain [ matcher=/api/**, filters =
    DisableEncodeUrlFilter, SecurityContextHolderFilter, HeaderWriterFilter,
    CorsFilter, BearerTokenAuthenticationFilter, ExceptionTranslationFilter,
    AuthorizationFilter ]
  SecurityFilterChain [ matcher=/**, filters =
    DisableEncodeUrlFilter, SecurityContextHolderFilter, CsrfFilter,
    UsernamePasswordAuthenticationFilter, ..., AuthorizationFilter ]
]
```

## Lifecycle

1. `DelegatingFilterProxy.doFilter()` → ищет бин `springSecurityFilterChain` ([[FilterChainProxy]])
2. вызывает `FilterChainProxy.doFilter()`
3. Подбирает SecurityFilterChain по URL и матчит список фильтров
4. Выполняет цепочку через `VirtualFilterChain`
5. После всех фильтров запрос передаёт в исходный `FilterChain` сервлета → дальше DispatcherServlet

friend:: [[SecurityFilterChain]]
friend:: [[SecurityContextHolder]]
friend:: [[SecurityConfig]]
friend:: [[DelegatingFilterProxy]]
child:: [[SecurityFilterChain]]
parent:: [[DelegatingFilterProxy]]