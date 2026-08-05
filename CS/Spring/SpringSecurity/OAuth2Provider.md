---
tags: [spring, security]
---

# OAuth2Provider

**OAuth2Provider** — реализация [[AuthProvider]] для аутентификации через внешних OAuth2-провайдеров: Google, GitHub, Facebook, Keycloak и др. Делегирует проверку токена внешнему серверу.

В Spring Security используются:
- `OAuth2LoginAuthenticationProvider` — для OAuth2 Login (userInfo + access_token)
- `OidcAuthorizationCodeAuthenticationProvider` — для OpenID Connect с id_token (JWT)
- `OidcLogoutAuthenticated` — для logout OIDC

## Поток OAuth2 Authorization Code

1. Пользователь нажимает "Login with Google" — фильтр `OAuth2AuthorizationRequestRedirectFilter` редиректит его на `authorization_endpoint` провайдера
2. Пользователь аутентифицируется на стороне провайдера и подтверждает доступ
3. Провайдер редиректит с `code` на `redirect_uri` приложения
4. Фильтр обменивает `code` на `access_token` (через `token_endpoint`)
5. Получает информацию о пользователе с `userInfo_endpoint`
6. Создаёт [[Principal]] с данными пользователя и authorities, сохраняет в [[SecurityContext]]

```
User        App            Google
 │           │                │
 │──click Login──→            │
 │           │──── redirect authorization_endpoint ──→│
 │           │←── authorization_required ──            │
 │── login at Google ─────────────────────────────────→│
 │           │←── authorization_code ──                │
 │           │──── POST token_endpoint (+code, secret)─→│
 │           │←── access_token ──                      │
 │           │──── GET userInfo_endpoint (+Bearer) ───→│
 │           │←── user info JSON ──                    │
 │           │
 │← authenticated, OAuth2AuthenticationToken в SecurityContext
```

## Конфигурация (Spring Boot)

```yaml
spring:
  security:
    oauth2:
      client:
        registration:
          google:
            client-id: ${GOOGLE_CLIENT_ID}
            client-secret: ${GOOGLE_CLIENT_SECRET}
            scope: profile, email
          github:
            client-id: ${GITHUB_CLIENT_ID}
            client-secret: ${GITHUB_CLIENT_SECRET}
            scope: read:user
```

```java
@Configuration
public class OAuth2Config {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(a -> a
                .requestMatchers("/", "/login**", "/error").permitAll()
                .anyRequest().authenticated())
            .oauth2Login(Customizer.withDefaults()) // включает OAuth2LoginAuthenticationFilter
            .logout(logout -> logout.logoutSuccessUrl("/"));
        return http.build();
    }
}
```

## Получение Principal-пользователя

```java
@GetMapping("/me")
public Map<String, String> me(@AuthenticationPrincipal OAuth2User principal) {
    return Map.of(
        "name", principal.getAttribute("name"),
        "email", principal.getAttribute("email"),
        "id",    principal.getAttribute("sub"));
}
```

## Доступ к access_token

```java
@GetMapping("/info")
public Map<String, Object> info(@RegisteredOAuth2AuthorizedClient OAuth2AuthorizedClient client) {
    String token = client.getAccessToken().getTokenValue();
    // используем его для вызова OAuth2-protected APIs
    return Map.of("token", token, "name", client.getPrincipalName());
}

// REST вызов с access_token к Google API:
String data = webClient.get()
    .uri("https://www.googleapis.com/userinfo")
    .header("Authorization", "Bearer " + token)
    .retrieve().bodyToMono(String.class).block();
```

## Связанные компоненты

- `ClientRegistration` — конфигурация OAuth2-клиента (clientId, clientSecret, endpoints). Регистрируется в `ClientRegistrationRepository`.
- `OAuth2AuthorizedClientService` — хранилище авторизованных клиентов с access_token.

parent:: [[AuthProvider]]
friend:: [[Principal]]
friend:: [[SecurityContext]]