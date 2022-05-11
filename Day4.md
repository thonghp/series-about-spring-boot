# Demo Cách Làm

## Mục lục nội dung

  - [1. Chức năng quản lý](#1-chức-năng-quản-lý)
    - [1.1 Get list](#11-get-list)
    - [1.2 Create new](#12-create-new)
    - [1.3 Encode password](#13-encode-password)
    - [1.4 Check Email Unique](#14-check-email-unique)
    - [1.5 Update](#15-update)
    - [1.6 Delete](#16-delete)
    - [1.7 Update status](#17-update-status)
    - [1.8 Upload image](#18-upload-image)
    - [1.9 Phân trang](#19-phân-trang)
    - [1.10 Sort](#110-sort)
    - [1.11 Search](#111-search)
    - [1.12 Refactor code](#112-refactor-code)
      - [1.12.1 Fragment](#1121-fragment)
      - [1.12.2 Cập nhật url khi thay đổi](#1122-cập-nhật-url-khi-thay-đổi)
    - [1.13 Xuất data ra file csv](#113-xuất-data-ra-file-csv)
    - [Xử lý redirect bằng jquery](#xử-lý-redirect-bằng-jquery)
  - [Upload ảnh](#upload-ảnh)

## 1. Chức năng quản lý 

**Demo quản lý user**

### 1.1 Get list 

- Lấy list data từ db đổ lên view 
    - Truy cập vào url **==>** load trang hiển thị tất cả thông tin 
        - **view ==> controller ==> service ==> repository**
- Cách làm
    - Tạo view root khi nhấn vào chuyển qua trang đổ list
    - Tạo layer **controller ==>** đổ data **==> GET ==>** thông qua **service** lấy list user từ db truyền vô **model** mang qua view
    - Tạo layer **service ==>** xử lý logic thông qua **repository** gọi method trả về list data 
    - Tạo layer **repository ==>** kế thừa các hàm có sẵn


```html
<!-- root -->
<a th:href="@{/users}" class="nav-link">Users</a>
<!-- root -->

<!-- users -->
<tr th:each="user: ${listUsers}">
    <td>[[${user.id}]]</td>
    <td>
        <span th:if="${user.photos == null}"></span>
        <img th:if="${user.photos != null}" th:src="@{${user.photosImagePath}}" style="width: 100px"/>
    </td>
    <td>[[${user.roles}]]</td>
    <td>
        <a th:if="${user.enabled == true}"></a>
        <a th:if="${user.enabled == false}"></a>
    </td>
</tr>
<!-- users -->
```    

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

### 1.2 Create new 

- Nhập form xong gửi data lên server 
    - Truy cập vào url **==>** trả về form **==>** nhập form **==>** submit về server
        - **view ==> controller ==> service ==> repository**
- Cách làm
    - Từ view root nhấn chuyển sang view form
    - Tạo layer **controller** để điều hướng Form **==> GET ==>** Tạo instance để lưu vô model khi qua view map với `th:object`
    - Nhập field xong submit **==> POST ==>** Lấy data sau đó thông qua **service** save xuống db **==>** redirect về list
    - Tạo layer **service ==>** xử lý logic thông qua **repository** gọi method save 

```html
<!-- user form  -->
<div>
    <h2>Manage Users | [[${pageTitle}]]</h2>
</div>
<form th:action="@{/users/save}" method="post" th:object="${user}">

    <input type="hidden" th:field="*{id}"/>

    <!-- map property với entity -->
    <input type="email" th:field="*{email}"/>
    
    <th:block th:each="role: ${listRoles}">
        <input type="checkbox" th:field="*{roles}" th:text="${role.name}" th:value="${role.id}"/>
        - <small>[[${role.description}]]</small>
        <br>
    </th:block>
</form>

<!-- user -->
<div th:if="${message != null}">
    <!-- nhận redirect từ server -->
    [[${message}]] 
</div>
```

```java
// controller
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

// service
public User save(User user) {
    return userRepository.save(user);
}
```

### 1.3 Encode password

- Encode mật khẩu
    - Nhập mật khẩu **==>** encode **==>** Lưu mật khẩu xuống db
        - **service ==> configuration**
- Cách làm
    1. Tạo **configuration ==>** định nghĩa cấu hình mã hoá là bcrypt
    2. Tạo layer **service ==>** encode password sau đó xử lý logic thông qua **repository** gọi method save 

```java
@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests().anyRequest().permitAll(); // allow public access to the admin app without authentication
    }
}

// service
@Autowired
private PasswordEncoder passwordEncoder;

private void encodePassword(User user) {
    String encodedPassword = passwordEncoder.encode(user.getPassword());
    user.setPassword(encodedPassword);
}

public User save(User user) {
    encodePassword(user);

    return userRepository.save(user);
}
```

### 1.4 Check Email Unique 

- Check email trước khi submit
    - Nhập email sau đó submit **==>** hợp lệ thì thành công còn không hợp lệ yêu cầu nhập lại
        - **view ==> rest controller ==> service ==> repository**
- Cách làm
    - Tại view xử lý check email submit bằng ajax
    - Tạo layer **rest controller ==>** lấy data submit **==> POST ==>** lấy id và tài khoản thông qua **service** check coi hợp lệ hay ko rồi gửi lại view để xử lý ajax
    - Tạo layer **service ==>** xử lý logic thông qua **repository** gọi method check email độc nhất
    - Tạo layer **repository ==>** viết câu query lấy thông tin thông qua email 

```html 
<form th:action="@{/users/save}" method="post" th:object="${user}" onsubmit="return checkEmailUnique(this);">
    <input type="hidden" th:field="*{id}"/>
    <input type="email" th:field="*{email}"/>

    <input type="submit" value="Save" class="btn btn-primary m-3"/>
</form>

<!-- modal -->
<div class="modal fade text-center" id="modalDialog">
    <div class="modal-dialog">
        <div class="modal-content">
            <div class="modal-header">
                <h4 class="modal-title" id="modalTitle">Warning</h4>
                <button type="button" class="close" data-dismiss="modal">&times;</button>
            </div>
            <div class="modal-body">
                <span id="modalBody"></span>
            </div>
        </div>
    </div>
</div>
<!-- modal -->

<script type="text/javascript">
function checkEmailUnique(form) {
    url = "[[@{/users/check_email}]]"; // url RestController
    userEmail = $("#email").val(); // lấy giá trị của #id
    userId = $("#id").val();
    // csrf token được tạo ra từ config spring security
    csrfValue = $("input[name='_csrf']").val(); 

    // các thuộc tính bên trong đúng với thuộc tính thật
    params = {id: userId, email: userEmail, _csrf: csrfValue}; 

    $.post(url, params, function (response) { // method post => url, data, callback
        if (response == "OK") {
            form.submit();
        } else if (response == "Duplicated") {
            showModalDialog("Warning", "Email " + userEmail + " đã tồn tại !")
        } else {
            showModalDialog("Error", "Unknown response from server");
        }
    }).fail(function () {
        showModalDialog("Error", "Could not connect to the server"); // this line runs when the above url is wrong
    });

    return false;
}

function showModalDialog(title, message) {
    $("#modalTitle").text(title);
    $("#modalBody").text(message);
    $("#modalDialog").modal();
}
</script>
```    

```java
@RestController
public class UserRestController {

    @Autowired
    private UserService userService;

    @PostMapping("/users/check_email")
    public String checkDuplicateEmail(@Param("id") Integer id, @Param("email") String email) {
        return userService.isEmailUnique(id, email) ? "OK" : "Duplicated";
    }
}

// service
public boolean isEmailUnique(Integer id, String email) {
    User userByEmail = userRepository.getUserByEmail(email);

    if (userByEmail == null) return true;

    boolean isCreatingNew = (id == null);

    if (isCreatingNew) {
        if (userByEmail != null) return false;
    } else {
        if (userByEmail.getId() != id) return false;
    }

    return true;
}

// repository
@Query("SELECT u FROM User u WHERE u.email = :email")
public User getUserByEmail(@Param("email") String email);
```

### 1.5 Update 

- Đổ data lên form **==>** chỉnh sửa **==>** submit data
    - Truy cập vào url **==>** load form chỉnh sửa **==>** chỉnh sừa **==>** submit về server
        - **view ==> controller ==> service ==> repository**
- Cách làm
    - Từ view quản lý **==>** điều hướng tới form muốn chỉnh sửa
    - Tạo layer **controller** đổ data lên form edit **==> GET ==>** lấy instance theo id thông qua service truyền vô model đổ vô view
        - Submit **==> POST ==>** Lấy data sau đó thông qua **service** save xuống db **==>** redirect về list

```html
<!-- users -->
<a th:href="@{'/users/edit/'+${user.id}}"></a>

<form th:action="@{/users/save}" method="post" th:object="${user}"
          onsubmit="return checkEmailUnique(this);">
    <input type="hidden" th:field="*{id}"/>

    <!-- áp dụng cho trường hợp save -->
    <input th:if="${user.id == null}" type="password" required th:field="*{password}"/>
    <!-- áp dụng cho trường hợp edit -->
    <input th:if="${user.id != null}" type="password" th:field="*{password}"/>
</form>
```        

```java
// controller
@GetMapping("/users/edit/{id}")
public String editUser(@PathVariable(name = "id") Integer id, Model model, RedirectAttributes redirectAttributes) {
    try {
        User user = userService.get(id);
        List<Role> listRoles = userService.listRoles();

        model.addAttribute("user", user);
        model.addAttribute("listRoles", listRoles);
        model.addAttribute("pageTitle", "Edit User (ID: " + id + ")");

        return "user_form";
    } catch (UserNotFoundException e) {
        redirectAttributes.addFlashAttribute("message", e.getMessage());

        return "redirect:/users";
    }
}

// service
public User get(Integer id) throws UserNotFoundException {
    try {
        return userRepository.findById(id).get();
    } catch (NoSuchElementException ex) {
        throw new UserNotFoundException("Could not find any user with ID" + id);
    }
}
```

### 1.6 Delete 

- Chọn user muốn xoá **==>** xoá
    - **view ==> controller ==> service ==> repository**
- Cách làm
    - Từ view quản lý **==>** chọn user muốn xoá 
    - Tạo layer **controller ==> GET ==>** xoá theo id thông qua service

```html
<a th:href="@{'/users/delete/'+${user.id}}" th:userId="${user.id}"></a>

<div class="modal fade text-center" id="confirmModal">
    <div class="modal-dialog">
        <div class="modal-content">
            <div class="modal-header">
                <h4 class="modal-title">Delete Confirmation</h4>
                <button type="button" class="close" data-dismiss="modal">&times;</button>
            </div>
            <div class="modal-body">
                <span id="confirmText"></span>
            </div>
            <div class="modal-footer">
                <a class="btn btn-success" href="" id="yesButton">Yes</a>
                <button type="button" class="btn btn-secondary" data-dismiss="modal">No</button>
            </div>
        </div>
    </div>
</div>

<script type="text/javascript">
    $(document).ready(function () {
        $(".link-delete").on("click", function (e) {
            e.preventDefault(); // Prevent a link from opening the URL

            link = $(this); // object jquery, can call all jQuery methods and functions but not DOM methods.
            userId = link.attr("userId"); //  Return the value of an attribute
            userId = link.attr("userId"); //  Return the value of an attribute

            $("#yesButton").attr("href", link.attr("href")); // set value of attribute
            $("#confirmText").text("Bạn có muốn xoá user " + userId + " không ?");
            $("#confirmModal").modal();
        });
    });
</script>
```

```java
@GetMapping("/users/delete/{id}")
public String deleteUser(@PathVariable(name = "id") Integer id, RedirectAttributes redirectAttributes) {
        try {
            userService.delete(id);

            redirectAttributes.addFlashAttribute("message", "User " + id + " xoá thành công !");
        } catch (UserNotFoundException e) {
            redirectAttributes.addFlashAttribute("message", e.getMessage());
        }

        return "redirect:/users";
    }

// service
public void delete(Integer id) throws UserNotFoundException {
    Long countById = userRepository.countById(id);

    if (countById == null || countById == 0) {
        throw new UserNotFoundException("Could not find any user with ID" + id);
    }

    userRepository.deleteById(id);
}

// repository
public Long countById(Integer id);
```

### 1.7 Update status 

- Chọn enable user muốn chỉnh **==>** nhấn kích hoạt 
    - **view ==> controller ==> service ==> repository**
- Cách làm
    - Từ view quản lý **==>** chọn user muốn tuỳ chỉnh enable 
    - Tạo layer **controller ==> GET ==>** tuỳ chỉnh enable theo id thông qua service

```html
<a th:if="${user.enabled == true}" th:href="@{'/users/'+${user.id}+'/enabled/false'}" ></a>
<a th:if="${user.enabled == false}" th:href="@{'/users/'+${user.id}+'/enabled/true'}"></a>
```

```java
// controller
@GetMapping("/users/{id}/enabled/{status}")
public String updateUserEnabledStatus(@PathVariable(name = "id") Integer id,
                                      @PathVariable(name = "status") boolean enabled,
                                      RedirectAttributes redirectAttributes) {
    userService.updateUpdateUserEnabledStatus(id, enabled);

    String status = enabled ? "enabled" : "disabled";
    String message = "The user id " + id + " has been " + status;

    redirectAttributes.addFlashAttribute("message", message);
    
    return "redirect:/users";
}

@Service
@Transactional
public class UserService {
    public void updateUserEnabledStatus(Integer id, boolean enabled) {
        userRepository.updateEnabledStatus(id, enabled);
    }
}

// repository
@Query("UPDATE User u SET u.enabled = ?2 WHERE u.id = ?1") // 1 with 2 is specifying the order of the attribute
@Modifying
public void updateEnabledStatus(Integer id, boolean enabled);
```

### 1.8 Upload image 

- Chọn ảnh **==>** gửi về server 
    - **view ==> controller ==> service ==> repository**
- Cách làm
    - Từ view quản lý **==>** load ảnh đã lưu trong thư mục 
    - Trong entity tạo hàm để map với view dùng để lấy đường dẫn
    - Trong form upload **==>** check size và hiển thị ảnh bằng javascript **==>** gửi về server
    - Tạo layer **controller** nhận data **==> POST ==>** xử lý thêm ảnh và xoá ảnh cũ nếu update 
    - Tạo file cấu hình cho phép truy cập tới file hệ thống để lấy ảnh hiển thị

- `enctype="multipart/form-data"` **==>** nói với browser khi gửi về có thể chứa file upload 
- `accept="image/png, image/jpeg"` **==>** hạn chế file upload gửi về 

```html
<!-- users -->
<span th:if="${user.photos == null}" class="fas fa-portrait fa-3x icon-silver"></span>
<img th:if="${user.photos != null}" th:src="@{${user.photosImagePath}}" style="width: 100px"/>

<!-- user_form -->
<form th:action="@{/users/save}" method="post" th:object="${user}"
    onsubmit="return checkEmailUnique(this);" enctype="multipart/form-data">

    <input type="hidden" th:field="*{photos}"/>
    <input type="file" id="fileImage" accept="image/png, image/jpeg" name="image"/>
    <img id="thumbnail" th:src="@{${user.photosImagePath}}"/>
</form>   

<script type="text/javascript">
$(document).ready(function () {
    // check kích thước đầu vào của ảnh 
    $("#fileImage").change(function () { // thực thi mỗi khi event kích hoạt
        // trả về file được chọn có thể lấy tên hoặc lấy size
        fileSize = this.files[0].size; 
        var mb = 1024 * 1024; // 1mb = 1024 x 1024 kb

        if (fileSize > mb) {
            this.setCustomValidity("Bạn phải chọn ảnh nhỏ hơn 1MB !");
            this.reportValidity();
        } else {
            // hiện lên dialog thông báo giống như chức năng required của input
            this.setCustomValidity("");
            showImageThumbnail(this);
        }
    });
});

function showImageThumbnail(fileInput) {
    var file = fileInput.files[0];
    // file reader dùng để đọc data trên máy tính user 1 cách không đồng bộ
    var reader = new FileReader(); 

    // onload kích hoạt khi readAsDataUrl,... đọc thành công
    reader.onload = function (e) {
        $("#thumbnail").attr("src", e.target.result);
    };

    /* 
    * đọc nội dung của blob hoặc file return url dạng string base 64
    * Sau đó lấy kết quả truyền vô scr img để hiển thị ảnh trên browser
    */
    reader.readAsDataURL(file); 
}
</script>
```

```java
// entity
@Transient
public String getPhotosImagePath() {
    if (id == null || photos == null) 
        return "/images/default-user.png";

    return "/user-photos/" + this.id + "/" + this.photos;
}

@PostMapping("/users/save")
public String saveUser(User user, RedirectAttributes redirectAttributes,
                      @RequestParam("image") MultipartFile multipartFile) throws IOException {
    if (!multipartFile.isEmpty()) {
        // lấy tên ảnh gửi từ multipart client lên
        String fileName = StringUtils.cleanPath(multipartFile.getOriginalFilename()); 
        user.setPhotos(fileName);
        User savedUser = userService.save(user);

        String uploadDir = "user-photos/" + savedUser.getId();

        FileUploadUtil.cleanDir(uploadDir); // remove ảnh cũ khi có ảnh mới
        FileUploadUtil.saveFile(uploadDir, fileName, multipartFile);
    } else {
        if (user.getPhotos().isEmpty())
            user.setPhotos(null);
        userService.save(user);
    }

    redirectAttributes.addFlashAttribute("message", "User đã được lưu thành công !");

    return "redirect:/users";
}

public class FileUploadUtil {
    public static void saveFile(String uploadDir, String fileName, MultipartFile multipartFile) throws IOException {
        Path uploadPath = Paths.get(uploadDir); // user-photo/1

        if (!Files.exists(uploadPath)) {
            Files.createDirectories(uploadPath);
        }

        try (InputStream inputStream = multipartFile.getInputStream()) {
            Path filePath = uploadPath.resolve(fileName); //user-photo\1\namhaminh.png
            // xử lý ảnh tải lên đưa vào thư mục cố định của system
            Files.copy(inputStream, filePath, StandardCopyOption.REPLACE_EXISTING);
        } catch (IOException e) {
            throw new IOException("Could not save file: " + fileName, e);
        }
    }

    public static void cleanDir(String dir) {
        Path dirPath = Paths.get(dir);

        try {
            Files.list(dirPath).forEach(file -> {
                if (!Files.isDirectory(file)) {
                    try {
                        Files.delete(file);
                    } catch (IOException e) {
                        System.out.println("Could not delete file: " + file);
                    }
                }
            });
        } catch (IOException e) {
            System.out.println("Could not list directory: " + dirPath);
        }
    }
}

@Configuration
public class MvcConfig implements WebMvcConfigurer {

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        String dirName = "user-photos";
        Path userPhotosDir = Paths.get(dirName);

        String userPhotosPath = userPhotosDir.toFile().getAbsolutePath();

        registry.addResourceHandler("/" + dirName + "/**").addResourceLocations("file:/" + userPhotosPath + "/");
    }
}
```

### 1.9 Phân trang 

- Chọn trang **==>** load list của trang đó
    - **view ==> controller ==> service ==> repository**
- Cách làm
    - Từ view quản lý **==>** chọn trang muốn hiển thị 
    - Tạo layer **controller** nhận thứ tự trang sau đó trả về **==> GET ==>** xử lý tổng trang, tổng phần tử,... 

```html
<div class="text-center m-1" th:if="${totalItems > 0}">
    <span>#[[${startCount}]] đến [[${endCount}]] của [[${totalItems}]] </span>
</div>
<div class="text-center m-1" th:unless="${totalItems > 0}">
    <span>Không tìm thấy User</span>
</div>
<div>
    <nav>
        <ul class="pagination justify-content-center">
            <li th:class="${currentPage > 1 ? 'page-item' : 'page-item disabled'}">
                <a th:href="@{/users/page/1}" class="page-link">First</a>
            </li>
            <li th:class="${currentPage > 1 ? 'page-item' : 'page-item disabled'}">
                <a th:href="@{'/users/page/'+${currentPage - 1}}" class="page-link">Previous</a>
            </li>
            <li th:class="${currentPage != i ? 'page-item' : 'page-item active'}"
                th:each="i : ${#numbers.sequence(1,totalPages)}">
                <a th:href="@{'/users/page/'+${i}}" class="page-link">[[${i}]]</a>
            </li>
            <li th:class="${currentPage < totalPages ? 'page-item' : 'page-item disabled'}">
                <a th:href="@{'/users/page/'+${currentPage + 1}}" class="page-link">Next</a>
            </li>
            <li th:class="${currentPage < totalPages ? 'page-item' : 'page-item disabled'}">
                <a th:href="@{'/users/page/'+${totalPages}}" class="page-link">Last</a>
            </li>
        </ul>
    </nav>
</div>
```

```java
// controller
// thay cho in ra list danh sách ban đầU
@GetMapping("/users") // http://localhost:8080/MTShopAdmin/users
public String listFirstPage(Model model) {
    return listByPage(1, model);
}

@GetMapping("/users/page/{pageNumber}")
public String listByPage(@PathVariable(name = "pageNumber") int pageNum, Model model) {
    Page<User> page = userService.listByPage(pageNum);
    List<User> listUsers = page.getContent(); // lấy nội dung của 1 page chuyển sang list

    long startElementOfPage = (pageNum - 1) * UserService.USER_PER_PAGE + 1;
    long endElementOfPage = startElementOfPage + UserService.USER_PER_PAGE - 1;

    if (endElementOfPage > page.getTotalElements()) {
        endElementOfPage = page.getTotalElements();
    }

    model.addAttribute("currentPage", pageNum);
    model.addAttribute("totalPages", page.getTotalPages());
    model.addAttribute("startCount", startElementOfPage);
    model.addAttribute("endCount", endElementOfPage);
    model.addAttribute("totalItems", page.getTotalElements());
    model.addAttribute("listUsers", listUsers);

    return "users";
}

// service
public static final int USER_PER_PAGE = 5;
public Page<User> listByPage(int pageNum) {
    // tại sao pageNum - 1 vì page nó hiểu trang đầu là 0
    Pageable pageable = PageRequest.of(pageNum - 1, USER_PER_PAGE); // chỉ định trang và giới hạn list của 1 trang

    return userRepository.findAll(pageable);
}

// repository
// api của spring data jpa
@Repository
public interface UserRepository extends PagingAndSortingRepository<User, Integer> {}
```

### 1.10 Sort 

- Chọn cột **==>** sắp xếp tăng giảm
    - **view ==> controller ==> service ==> repository**
- Cách làm
    - Từ view quản lý **==>** chọn cột muốn sắp xếp 
    - Tạo layer **controller** sắp xếp theo cột **==> GET ==>** xử lý sắp xếp và chuyển trang cũng áp dụng

```html
<!-- cột -->
<a th:if="${sortField != 'id'}" th:class="text-white"
   th:href="@{'/users/page/' + ${currentPage} + '?sortField=id&sortType=' + ${sortType}}">
    ID
</a>
<a th:if="${sortField == 'id'}" th:class="text-white"
   th:href="@{'/users/page/' + ${currentPage} + '?sortField=id&sortType=' + ${reverseSortType}}">
    ID
</a>
<span th:if="${sortField == 'id'}"
      th:class="${sortType == 'asc' ? 'fas fa-sort-up' : 'fas fa-sort-down'}">
</span>

<!-- phân trang -->
<li th:class="${currentPage > 1 ? 'page-item' : 'page-item disabled'}">
    <a th:href="@{'/users/page/1?sortField=' + ${sortField} + '&sortType=' + ${sortType}}"
       class="page-link">First
    </a>
</li>
<li th:class="${currentPage > 1 ? 'page-item' : 'page-item disabled'}">
    <a th:href="@{'/users/page/' + ${currentPage - 1} + '?sortField=' + ${sortField} + '&sortType=' + ${sortType}}"
       class="page-link">Previous
    </a>
</li>
<li th:class="${currentPage != i ? 'page-item' : 'page-item active'}"
    th:each="i : ${#numbers.sequence(1,totalPages)}">
    <a th:href="@{'/users/page/' + ${i} + '?sortField=' + ${sortField} + '&sortType=' + ${sortType}}"
       class="page-link">[[${i}]]
    </a>
</li>
<li th:class="${currentPage < totalPages ? 'page-item' : 'page-item disabled'}">
    <a th:href="@{'/users/page/' + ${currentPage + 1} + '?sortField=' + ${sortField} + '&sortType=' + ${sortType}}"
       class="page-link">Next
    </a>
</li>
<li th:class="${currentPage < totalPages ? 'page-item' : 'page-item disabled'}">
    <a th:href="@{'/users/page/' + ${totalPages} + '?sortField=' + ${sortField} + '&sortType=' + ${sortType}}"
       class="page-link">Last
    </a>
</li>
```

```java
@GetMapping("/users") 
public String listFirstPage(Model model) {
    return listByPage(1, model, "firstName", "asc");
}

@GetMapping("/users/page/{pageNumber}")
public String listByPage(@PathVariable(name = "pageNumber") int pageNum, Model model,
                         @Param("sortField") String sortField, @Param("sortType") String sortType) {
    Page<User> page = userService.listByPage(pageNum, sortField, sortType);
    List<User> listUsers = page.getContent();

    long startElementOfPage = (pageNum - 1) * UserService.USER_PER_PAGE + 1;
    long endElementOfPage = startElementOfPage + UserService.USER_PER_PAGE - 1;

    if (endElementOfPage > page.getTotalElements()) {
        endElementOfPage = page.getTotalElements();
    }

    String reverseSortType = "asc".equals(sortType) ? "desc" : "asc";

    model.addAttribute("currentPage", pageNum);
    model.addAttribute("totalPages", page.getTotalPages());
    model.addAttribute("startCount", startElementOfPage);
    model.addAttribute("endCount", endElementOfPage);
    model.addAttribute("totalItems", page.getTotalElements());
    model.addAttribute("listUsers", listUsers);
    model.addAttribute("sortField", sortField);
    model.addAttribute("sortType", sortType);
    model.addAttribute("reverseSortType", reverseSortType);

    return "users";
}

// service
public Page<User> listByPage(int pageNum, String sortField, String sortType) {
    Sort sort = Sort.by(sortField);
    sort = sortType.equals("asc") ? sort.ascending() : sort.descending();

    Pageable pageable = PageRequest.of(pageNum - 1, USER_PER_PAGE, sort);

    return userRepository.findAll(pageable);
}
```

### 1.11 Search

- Nhập keyword **==>** tìm kiếm
    - **view ==> controller ==> service ==> repository**
- Cách làm
    - Từ view quản lý **==>** nhập keyword muốn kiếm
    - Tạo layer **controller** tìm kiếm **==> GET ==>** trả về thông tin hợp lệ

```html
<!-- thanh tìm kiếm -->
<form th:action="@{/users/page/1}">
    <input type="hidden" name="sortField" th:value="${sortField}"/>
    <input type="hidden" name="sortType" th:value="${sortType}"/>

    Tìm Kiếm:
    <input type="search" name="keyword" class="form-control" required placeholder="Search"/>
    <input type="submit" value="Search"/>
    <input type="button" value="Clear" onclick="clearFilter()"/>
</form>

<!-- áp dụng tìm kiếm lên từng cột khi thay đổi thì cột sẽ thay đổi -->
<a th:if="${sortField != 'id'}" th:class="text-white"
   th:href="@{'/users/page/' + ${currentPage} + '?sortField=id&sortType=' + ${sortType} +
   ${keyword != null ? '&keyword=' + keyword : ''}}">
    ID
</a>
<a th:if="${sortField == 'id'}" th:class="text-white"
   th:href="@{'/users/page/' + ${currentPage} + '?sortField=id&sortType=' + ${reverseSortType} +
   ${keyword != null ? '&keyword=' + keyword : ''}}">
    ID
</a>
<span th:if="${sortField == 'id'}"
      th:class="${sortType == 'asc' ? 'fas fa-sort-up' : 'fas fa-sort-down'}">
</span>

<!-- phân trang -->
<div th:if="${totalPages > 0}">
    <nav>
        <ul class="pagination justify-content-center">
            <li th:class="${currentPage > 1 ? 'page-item' : 'page-item disabled'}">
                <a th:href="@{'/users/page/1?sortField=' + ${sortField} + '&sortType=' + ${sortType} +
                   ${keyword != null ? '&keyword=' + keyword : ''}}" class="page-link">
                    First
                </a>
            </li>
            <li th:class="${currentPage > 1 ? 'page-item' : 'page-item disabled'}">
                <a th:href="@{'/users/page/' + ${currentPage - 1} + '?sortField=' + ${sortField} + '&sortType=' + ${sortType} +
                   ${keyword != null ? '&keyword=' + keyword : ''}}" class="page-link">
                    Previous
                </a>
            </li>
            <li th:class="${currentPage != i ? 'page-item' : 'page-item active'}"
                th:each="i : ${#numbers.sequence(1,totalPages)}">
                <a th:href="@{'/users/page/' + ${i} + '?sortField=' + ${sortField} + '&sortType=' + ${sortType} +
                   ${keyword != null ? '&keyword=' + keyword : ''}}"
                   class="page-link">[[${i}]]
                </a>
            </li>
            <li th:class="${currentPage < totalPages ? 'page-item' : 'page-item disabled'}">
                <a th:href="@{'/users/page/' + ${currentPage + 1} + '?sortField=' + ${sortField} + '&sortType=' + ${sortType} +
                   ${keyword != null ? '&keyword=' + keyword : ''}}"
                   class="page-link">Next
                </a>
            </li>
            <li th:class="${currentPage < totalPages ? 'page-item' : 'page-item disabled'}">
                <a th:href="@{'/users/page/' + ${totalPages} + '?sortField=' + ${sortField} + '&sortType=' + ${sortType} +
                   ${keyword != null ? '&keyword=' + keyword : ''}}"
                   class="page-link">Last
                </a>
            </li>
        </ul>
    </nav>
</div>
```

```java
@GetMapping("/users") // http://localhost:8080/MTShopAdmin/users
public String listFirstPage(Model model) {
    return listByPage(1, model, "firstName", "asc", null);
}

@GetMapping("/users/page/{pageNumber}")
public String listByPage(@PathVariable(name = "pageNumber") int pageNum, Model model,
                         @Param("sortField") String sortField, @Param("sortType") String sortType,
                         @Param("keyword") String keyword) {
    Page<User> page = userService.listByPage(pageNum, sortField, sortType, keyword);
    List<User> listUsers = page.getContent();

    long startElementOfPage = (pageNum - 1) * UserService.USER_PER_PAGE + 1;
    long endElementOfPage = startElementOfPage + UserService.USER_PER_PAGE - 1;

    if (endElementOfPage > page.getTotalElements()) {
        endElementOfPage = page.getTotalElements();
    }

    String reverseSortType = "asc".equals(sortType) ? "desc" : "asc";

    model.addAttribute("currentPage", pageNum);
    model.addAttribute("totalPages", page.getTotalPages());
    model.addAttribute("startCount", startElementOfPage);
    model.addAttribute("endCount", endElementOfPage);
    model.addAttribute("totalItems", page.getTotalElements());
    model.addAttribute("listUsers", listUsers);
    model.addAttribute("sortField", sortField);
    model.addAttribute("sortType", sortType);
    model.addAttribute("reverseSortType", reverseSortType);
    model.addAttribute("keyword", keyword);

    return "users";
}

// service
public Page<User> listByPage(int pageNum, String sortField, String sortType, String keyword) {
    Sort sort = Sort.by(sortField);
    sort = sortType.equals("asc") ? sort.ascending() : sort.descending();

    Pageable pageable = PageRequest.of(pageNum - 1, USER_PER_PAGE, sort);

    if (keyword != null)
        return userRepository.findAll(keyword, pageable);

    return userRepository.findAll(pageable);
}

@Query("SELECT u FROM User u WHERE CONCAT(u.id, ' ', u.email, ' ', u.firstName, ' ', u.lastName) LIKE %?1%")
public Page<User> findAll(String keyword, Pageable pageable);
```

### 1.12 Refactor code 

#### 1.12.1 Fragment 

- Tạo file `fragments.html`
- Tìm điểm chung ở đây điểm chung chính là trang hiện tại và tên các nút

```html
<!-- file gốc ở đây cái nút với số thứ tự trang hiện tại lặp lại -->
<a th:href="@{'/users/page/' + ${totalPages} + '?sortField=' + ${sortField} + '&sortType=' + ${sortType} + ${keyword != null ? '&keyword=' + keyword : ''}}">
    Last
</a>

<!-- trang fragments -->
<a th:fragment="page_link(currentPage, label)" class="page-link"
   th:href="@{'/users/page/' + ${currentPage} + '?sortField=' + ${sortField} + '&sortType=' + ${sortType} + ${keyword != null ? '&keyword=' + keyword : ''}}">
    [[${label}]]
</a>

<!-- sửa đoạn gốc đã được thay thế -->
<a th:replace="fragments :: page_link(${totalPages}, 'Last')"></a>
```

#### 1.12.2 Cập nhật url khi thay đổi 

```java
@PostMapping("/users/save")
public String saveUser(User user, RedirectAttributes redirectAttributes, @RequestParam("image") MultipartFile multipartFile) throws IOException {
    if (!multipartFile.isEmpty()) {
        String fileName = StringUtils.cleanPath(multipartFile.getOriginalFilename());
        user.setPhotos(fileName);

        User savedUser = userService.save(user);

        String uploadDir = "images/user-photos/" + savedUser.getId();

        FileUploadUtil.cleanDir(uploadDir); // remove ảnh cũ khi có ảnh mới
        FileUploadUtil.saveFile(uploadDir, fileName, multipartFile);
    } else {
        if (user.getPhotos().isEmpty())
            user.setPhotos(null);
        userService.save(user);
    }

    redirectAttributes.addFlashAttribute("message", "User đã được lưu thành công !");

     //   return "redirect:/users"; // phiên bản cũ
    // sau khi thêm hoặc xoá sẽ hiện lên trang chỉ có 1 user mới này thay vì hiện list như cũ 
    String firstPartOfEmail = user.getEmail().split("@")[0];
    return "redirect:/users/page/1?sortField=id&sortType=asc&keyword=" + firstPartOfEmail; 
}
```

### 1.13 Xuất data ra file csv 

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