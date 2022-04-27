## 1. Quản lý 

**Demo quản lý user**

### 1.1 Get list 

Đổ list data lên  **view ==> controller ==> service ==> repository**

- Demo controller đổ tới url `/users` từ file `users.html`
    - truy xuất data user **từ db** thông qua **repository** trả về **service** xử lý logic sau đó sử dụng trong controller để **đổ vô model** mang **qua html**

```java
@Controller
public class UserController {

    @Autowired
    private UserService userService;

    @GetMapping("/users") // url
    public String listAll(Model model) {
        List<User> listUsers = userService.listAll();

        // = request.setAttribute() of HttpServletRequest
        model.addAttribute("listUsers", listUsers); 

        return "users"; // users.html
    }
}

@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;

    public List<User> listAll() {
        return (List<User>) userRepository.findAll();
    }
}

@Repository
public interface UserRepository extends CrudRepository<User, Integer> {}
```

```html
<tr th:each="user: ${listUsers}">
    <td>[[${user.id}]]</td>
    <td>
        <span th:if="${user.photos == null}" class="fas fa-portrait fa-3x icon-silver"></span>
        <img th:if="${user.photos != null}" th:src="@{${user.photosImagePath}}" style="width: 100px"/>
    </td>
    <td>[[${user.roles}]]</td>
    <td>
        <a th:if="${user.enabled == true}" class="fas fa-check-circle fa-2x icon-green"></a>
        <a th:if="${user.enabled == false}" class="fas fa-check-circle fa-2x icon-dark"></a>
    </td>
</tr>
```

### 1.2 Create new 

- GET **==>** Tạo instance user để map với `th:object` bên thymeleaf
- POST **==>** Lấy data từ form về **==>** hiển thị bên user và kèm theo câu thông báo xác nhận save thành công 

```java
@GetMapping("/users/new")
public String newUser(Model model) {
    User user = new User();
    List<Role> listRoles = userService.listRoles();

    user.setEnabled(true);

    model.addAttribute("user", user);
    model.addAttribute("listRoles", listRoles);
    model.addAttribute("pageTitle", "Create New User");

    return "user_form";
}

@PostMapping("/users/save")
public String saveUser(User user, RedirectAttributes redirectAttributes) {
    userService.save(user);

    redirectAttributes.addFlashAttribute("message", "User đã được lưu thành công !");

    return "redirect:/users";
}
```

```html
<!-- user form  -->
<div>
    <h2>Manage Users | [[${pageTitle}]]</h2>
</div>
<form th:action="@{/users/save}" method="post" style="max-width: 700px; margin: 0 auto" th:object="${user}">
        <input type="hidden" th:field="*{id}"/>
        <div class="border border-secondary rounded p-3">
            <div class="form-group row">
                <label class="col-sm-4 col-form-label">E-mail</label>
                <div class="col-sm-8">
                    <!-- map attribute with entity -->
                    <input type="email" class="form-control" required minlength="8" maxlength="128"
                           th:field="*{email}"/>
                </div>
            </div>
            <div class="form-group row">
                <label class="col-sm-4 col-form-label">Roles:</label>
                <div class="col-sm-8 mt-2">
                    <th:block th:each="role: ${listRoles}">
                        <input type="checkbox" th:field="*{roles}" th:text="${role.name}" th:value="${role.id}"
                               class="mt-2 mr-2"/>
                        - <small>[[${role.description}]]</small>
                        <br>
                    </th:block>
                </div>
            </div>
            <div class="text-center">
                <input type="submit" value="Save" class="btn btn-primary m-3"/>
                <input type="button" value="Cancel" class="btn btn-secondary" id="btnCancel">
            </div>
        </div>
</form>
<!-- user  -->
<div th:if="${message != null}" class="alert alert-success text-center">
        [[${message}]]
</div>
```

### 1.3 Encode password

- Sử dụng bcrypt để encode password trong spring security

### Xử lý redirect bằng jquery

```javascript
<div class="text-center">
    <input type="submit" value="Save" class="btn btn-primary m-3"/>
    <input type="button" value="Cancel" class="btn btn-secondary" id="btnCancel">
</div>

// window.location = location object 
<script type="text/javascript">
    $(document).ready(function () {
        $("#btnCancel").on("click", function () {
            window.location = "[[@{/users}]]";
        });
    });
</script>
```

## Upload ảnh

**Yêu cầu**
- Hình ảnh là tuỳ chọn
- Ảnh sẽ được lưu vô hệ thống
- Tên ảnh lưu xuống db
- Hiện ảnh trong list
- File size < 1MB
- Xoá ảnh cũ khi cập nhật 

**Cơ chế**

- Không **==>** set ảnh mặc định
- Có **==>** Lưu ảnh 

**Demo**

```html 

```

```java

```