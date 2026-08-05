---
tags: [spring, annotation]
---

# @Controller

**`@Controller`** — специализация [[@Component]] для веб-слоя в Spring MVC. Помечает классы, обрабатывающие HTTP-запросы.

## Сфера действия

Управляется [[Servlet WebApplicationContext]] — дочерним контекстом в иерархии Spring, который содержит только веб-компоненты (контроллеры, handler mappings, view resolvers).

## Привязка к URL

Методы контроллера привязываются к URL через [[@RequestMapping]] или его специализации:
- `@GetMapping` — для обработки GET-запросов
- `@PostMapping` — для POST
- `@PutMapping`, `@DeleteMapping`, `@PatchMapping` — для остальных HTTP-методов

## Возврат результата

Возвращаемое значение метода (логическое имя представления или объект `ModelAndView`) обрабатывается [[ViewResolver]] для рендеринга [[Web page]].

Для REST-эндпойнтов вместо `@Controller` используют `@RestController` (= `@Controller` + `@ResponseBody`), который напрямую сериализует ответ в JSON/XML без ViewResolver.

## Пример: MVC-контроллер

```java
@Controller
@RequestMapping("/users")
public class UserController {

    private final UserService service;
    public UserController(UserService s) { this.service = s; }

    // GET /users/{id} → рендеринг JSP "users/detail"
    @GetMapping("/{id}")
    public String show(@PathVariable Long id, Model model) {
        model.addAttribute("user", service.findById(id));
        return "users/detail";   // ViewResolver → /WEB-INF/views/users/detail.jsp
    }

    // GET /users → список
    @GetMapping
    public String list(Model model) {
        model.addAttribute("users", service.findAll());
        return "users/list";
    }

    // POST /users → создать
    @PostMapping
    public String create(@ModelAttribute UserForm form) {
        service.save(form);
        return "redirect:/users";
    }
}
```

## Пример: REST-контроллер

```java
@RestController
@RequestMapping("/api/users")
public class UserRestController {
    private final UserService service;
    public UserRestController(UserService s) { this.service = s; }

    @GetMapping("/{id}")
    public UserDto get(@PathVariable Long id) {
        return service.findDtoById(id);      // @ResponseBody → Jackson → JSON
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public UserDto create(@RequestBody @Valid UserCreateRequest req) {
        return service.create(req);          // @RequestBody из JSON
    }

    @PutMapping("/{id}")
    public UserDto update(@PathVariable Long id,
                          @RequestBody UserUpdateRequest req) {
        return service.update(id, req);
    }

    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void delete(@PathVariable Long id) { service.delete(id); }
}
```

## В DispatcherServlet

`@Controller`-класс обёртывается в [[Handle Object]] внутри [[DispatcherServlet]] при обработке запроса.

## Аннотации параметров методов

| Аннотация | Откуда | Пример |
|-----------|--------|--------|
| `@PathVariable` | часть URL | `/users/{id}` → id=42 |
| `@RequestParam` | query string | `?page=1` |
| `@RequestBody` | тело запроса (JSON) | POST `/api/users` |
| `@RequestHeader` | HTTP-заголовок | `Authorization` |
| `@CookieValue` | cookie | `JSESSIONID` |
| `@ModelAttribute` | form data | POST `/users` |
| `@AuthenticationPrincipal` | текущий Principal | данные пользователя |

friend:: [[Web page]]
friend:: [[ViewResolver]]
friend:: [[@RequestMapping]]
friend:: [[Servlet WebApplicationContext]]
friend:: [[Handle Object]]
friend:: [[DispatcherServlet]]
parent:: [[@Component]]