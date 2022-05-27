# Thymeleaf

## Mục lục nội dung 

  - [1. URL](#1-url)
  - [2. Thymeleaf Standard Expression](#2-thymeleaf-standard-expression)
  - [3. Authentication](#3-authentication)
  - [4. Fragment](#4-fragment)
  
## 1. URL           

- Thuộc tính sử dụng **==>** `th:src`, `th:href`, `th:action`
- Cú pháp **==>** `"@{/...}"` 
    - Trang đầu tiên **==>** `@{/}`
    - Nối url `"@{'/users/edit/'+${user.id}}"` **==>** users/edit/1
    - absolute **==>** `@{https://frontbackend.com/tag/thymeleaf}`
    - context-relative **==>** `@{/images/logo.png}"`
    - Url của class **==>** `@{${user.photosImagePath}}`

## 2. Thymeleaf Standard Expression

- Thuộc tính thường sử dụng trong form **==>** `th:action`, `th:object`, `th:field`, `th:block`, `th:each`, `th:text`, `th:value`
- Cú pháp
    - `th:action` **==>** điều hướng tới server xử lý submit form, tương đương với `th:attr="action="`
        - Vd: `th:action="@{/users/save}"`
    - `th:object` **==>** map tới entity, tương đương với `session.entity`
        - Vd: `th:object="${user}"` **==> user** là `Model` 
    - `th:field` **==>** thuộc tính của object map với entity, tương đương với `session.entity.properties`
        - Vd: `th:field="*{email}"`
    - `th:block` **==>** tạo ra 1 vùng chứa để thực thi và sau khi xong sẽ tự mất, thường đi với `th:each`
        - Vd: `<th:block th:each="role: ${listRoles}"></th:block>`
    - `th:text` **==>** Đổ data dưới dạng text 
        - tương đương với `[[${role.description}]]` - **Nội tuyến**
    - `th:value` **==>** đổ giá trị thường trong thẻ input   
- `th:if` **==>** Câu điều kiện 
    - Vd: `th:if="${message != null}"` **==> message** có thể là `RedirectAttributes` hoặc `Model`
- `th:unless` **==>** phủ định của `th:if`
    - Vd: `th:if="${message != null}"` **==>** `th:unless="${message != null}"`
- `th:each` **==>** Vòng lặp
    - Vd: `th:each="role: ${listRoles}"` **==>** `[[${role.name}]]` **==> listRoles** thường là **Model** 
    - Vd: `th:each="i : ${#numbers.sequence(1,5)}">` **==>** `[[${i}]]` **==>** i chạy từ 1 đến 5
    - Vd: `th:each="row,rowStat : ${customers}"` **==>** rowStat là biến trạng thái
- `th:custom` **==>** custom là thuộc tính tuỳ chỉnh không nằm trong html 
    - Vd: `th:userId="${user.id}"` 
- `th:remove` **==>** xoá thẻ tuỳ chỉnh
    - `th:remove="all"` **==>** Xoá thẻ chứa và tất cả thẻ con
    - `th:remove="body"` **==>** Không xoá thẻ chứa nhưng xoá tất cả thẻ con
    - `th:remove="tag"` **==>** Xoá thẻ chứa nhưng không xoá thẻ con
    - `th:remove="all-but-first"` **==>** Xoá thẻ chứa và tất cả thẻ con
    - `th:remove="none"` **==>** Không làm gì cả

## 3. Authentication      

- `xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity5"` khai báo trên thẻ html
- `sec:authentication="principal.fullname"` **==>** truy cập authentication object 
    - **principal.fullname** với fullname là method getFullname nằm trong file implements **UserDetails**

### 4. Fragment

Dùng để tách code ra phục vụ cho việc tái sử dụng 

- `th:fragment` **==>** Chỉ định là fragment 
    - Vd: fragment ko có tham số `th:fragment="column_link"` và có tham số `th:fragment="column_link(fieldName)"`
- `th: replace` **==>** thay thế thành thẻ fragment đã chỉ định    

```html
<th th:fragment="column_link(fieldName, columnLabel, removeTag)" th:remove="${removeTag}">
    <a th:class="text-white" th:href="@{'/users/page/' + ${currentPage} + '?sortField=' + ${fieldName} + '&sortType=' +
    ${sortField != fieldName ? sortType : reverseSortType} + '&keyword=' + ${keyword != null ? keyword : ''}}">
        [[${columnLabel}]]
    </a>
    <span th:if="${sortField == fieldName}"
          th:class="${sortType == 'asc' ? 'fas fa-sort-up' : 'fas fa-sort-down'}">
    </span>
</th>

<th th:replace="fragments :: column_link('lastName', 'Họ', 'none')"/>
```
