---
tags: [spring, security]
---

# SecurityConfig

**`SecurityConfig`** — класс конфигурации Spring Security, помеченный [[@Configuration]]. Здесь декларативно описывается вся безопасность приложения через специальный DSL.

## Что настраивает

- **[[SecurityFilterChain]]** через `HttpSecurity` DSL: какие URL защищены, форма логина (form-login), logout, CSRF, CORS, OAuth2/JWT
- **[[PasswordEncoder]]** — бин для хеширования паролей (например, `BCryptPasswordEncoder`)
- **[[AuthManager]]** (AuthenticationManager) — bean через `AuthenticationManagerBuilder`, определяет [[AuthProvider]]
- **[[UserDetailsService]]** — источник данных о пользователях
- **[[FilterChainProxy]]** — Spring создаёт его автоматически на основе `SecurityFilterChain`

Несколько `SecurityFilterChain` позволяют задать разные правила для разных URL-паттернов (например, отдельный набор фильтров для `/api/**`).

## Минимальная конфигурация (Spring Security 6)

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/login", "/css/**", "/register").permitAll()
                .requestMatchers("/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated())
            .formLogin(form -> form
                .loginPage("/login")
                .defaultSuccessUrl("/"))
            .logout(logout -> logout
                .logoutUrl("/logout")
                .logoutSuccessUrl("/login"))
            .csrf(Customizer.withDefaults());
        return http.build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

## С UserDetailsService и AuthProvider

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    private final UserDetailsService uds;
    private final PasswordEncoder enc;

    public SecurityConfig(UserDetailsService uds, PasswordEncoder enc) {
        this.uds = uds; this.enc = enc;
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(a -> a
                .requestMatchers("/api/public/**").permitAll()
                .anyRequest().authenticated())
            .httpBasic(Customizer.withDefaults());
        return http.build();
    }

    @Bean
    public AuthenticationProvider authProvider() {
        DaoAuthenticationProvider p = new DaoAuthenticationProvider();
        p.setUserDetailsService(uds);
        p.setPasswordEncoder(enc);              // BCrypt
        return p;
    }

    @Bean
    public AuthenticationManager authManager(HttpSecurity http) throws Exception {
        return http.getSharedObject(AuthenticationManagerBuilder.class)
                   .authenticationProvider(authProvider())
                   .build();
    }
}
```

## REST + JWT (пример схема)

```java
@Configuration
@EnableWebSecurity
public class JwtSecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http, JwtAuthFilter jwtFilter) throws Exception {
        http
            .csrf(c -> c.disable())
            .sessionManagement(s -> s.sessionCreationPolicy(STATELESS))
            .authorizeHttpRequests(a -> a
                .requestMatchers("/auth/login", "/auth/refresh").permitAll()
                .anyRequest().authenticated())
            .addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class);
        return http.build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() { return new BCryptPasswordEncoder(); }
}
```

## Несколько цепочек фильтров (правило: первый подходящий)

```java
@Bean
@Order(1)
public SecurityFilterChain apiChain(HttpSecurity http) throws Exception {
    http
        .securityMatcher("/api/**")
        .csrf(c -> c.disable())
        .authorizeHttpRequests(a -> a.anyRequest().authenticated())
        .oauth2ResourceServer(o -> o.jwt(Customizer.withDefaults()));
    return http.build();
}

@Bean
@Order(2)
public SecurityFilterChain webChain(HttpSecurity http) throws Exception {
    http
        .authorizeHttpRequests(a -> a
            .requestMatchers("/login", "/css/**").permitAll()
            .anyRequest().authenticated())
        .formLogin(Customizer.withDefaults());
    return http.build();
}
```

friend:: [[@Configuration]]
friend:: [[SecurityFilterChain]]
friend:: [[FilterChainProxy]]
friend:: [[PasswordEncoder]]
friend:: [[AuthManager]]
friend:: [[UserDetailsService]]
friend:: [[DAOAuthProvider]]
friend:: [[AuthManager]]
parent:: [[@Configuration]]