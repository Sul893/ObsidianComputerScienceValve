---
tags: [web]
---

# Web Page

**Веб-страница (Web Page)** — документ, доступный по HTTP, который возвращается [[Servlet]] в ответ на HTTP-запрос. Стандартный отклик веб-приложения клиенту (браузеру).

## Жизненный цикл

```
HTTP-запрос → ServletContainer → Servlet
           → бизнес-логика, подготовка модели данных
           → выбор шаблона (по имени представления)
           → разрешение шаблона → реальный путь (префикс/суффикс)
           → рендеринг шаблона с моделью → HTML
           → контейнер отправляет HTML обратно клиенту
```

## Технологии представления

- **JSP (Java Server Pages)** — традиционный шаблонизатор Jakarta EE. Компилируется в сервлет и исполняется контейнером. По умолчанию pages лежат в `/WEB-INF/`.
- **HTML** — статические страницы, отдаваемые напрямую. Без серверной логики, кроме ресурсов.
- **Thymeleaf** — современный серверный шаблонизатор. Поддерживает "natural templating" (шаблон остаётся валидным HTML).
- **FreeMarker** — альтернативный шаблонизатор с мощным expression syntax.

## Пример: JSP + Servlet

### Servlet (без фреймворков)

```java
@WebServlet("/users/*")
public class UserController extends HttpServlet {
    private final UserService service = new UserService();

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {
        Long id = Long.valueOf(req.getPathInfo().substring(1));
        req.setAttribute("user", service.findById(id));
        req.getRequestDispatcher("/WEB-INF/views/users/detail.jsp")
           .forward(req, resp);  // делегируем рендеринг JSP
    }
}
```

### JSP `/WEB-INF/views/users/detail.jsp`

```jsp
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<!DOCTYPE html>
<html>
<head><title>User Detail</title></head>
<body>
    <h1>${user.name}</h1>
    <p>Email: ${user.email}</p>
    <c:if test="${not empty user.bio}">
        <p>${user.bio}</p>
    </c:if>
</body>
</html>
```

## Разрешение шаблона (view resolver-стиль)

MVC-фреймворки поверх сервлетов обычно вводят понятие *view resolver* — компонента, разрешающая логическое имя представления в конкретную Web Page:

```
Servlet returns: "users/detail"
                  ↓ view resolver
                  /WEB-INF/views/users/detail.jsp
                  ↓ рендеринг
                  HTML байты
                  ↓
                  Tomcat отправляет
```

Логическое имя `users/detail` + префикс `/WEB-INF/views/` + суффикс `.jsp` → реальный путь внутри WAR.

friend:: [[Servlet]]