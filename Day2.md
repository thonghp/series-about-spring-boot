# Thymeleaf

## Mục lục nội dung 

  - [1. Overview](#1-overview)
  - [2. Expression](#2-expression)
  - [3. Fragment](#3-fragment)

## 1. Overview

- Là một kiểu sever side rendering
- Là một view template engine
- Data + view template ==> HTML 

## 2. Expression

- `${}` **==>** thay giá trị của biến 
- `*{}` **==>** dùng trong form khi data binding
- `#{}` **==>** message expression, biểu thức thay chuỗi đa ngôn ngữ từ file resource.
- `@{}` **==>** chỉ thị đường dẫn
    - thường được sử dụng bởi **==>** `th:src`, `th:href`, `th:action`
        - Trang đầu tiên **==>** `@{/}`
        - Nối url `"@{'/users/edit/'+${user.id}}"` **==>** users/edit/1
        - absolute **==>** `@{https://frontbackend.com/tag/thymeleaf}`
        - context-relative **==>** `@{/images/logo.png}"`
        - Url của class **==>** `@{${user.photosImagePath}}`
- `~{}` **==>** fragement expression
- `th:action` **==>** điều hướng tới server xử lý submit form, tương đương với `th:attr="action="`
    - Vd: `th:action="@{/users/save}"`
- `th:object` **==>** map tới entity, tương đương với `session.entity`
    - Vd: `th:object="${user}"` **==> user** là `Model` 
- `th:field` **==>** thuộc tính của object map với entity, tương đương với `session.entity.properties`
    - Vd: `th:field="*{email}"`
- `th:block` **==>** tạo ra 1 vùng chứa để thực thi và sau khi xong sẽ tự mất, thường đi với `th:each`
    - Vd: `<th:block th:each="role: ${listRoles}"></th:block>`
- `th:text` **==>** Đổ một đoạn text lên view 
    - tương đương với `[[${role.description}]]` - **Nội tuyến**
- `th:utext` **==>** Đổ một đoạn text có chứa thẻ HTML, CSS vào view 
    - `model.addAttribute("message", "<span style='color:red'>HTML</span>");`
- `th:value` **==>** đổ giá trị thường trong thẻ input   
- `th:if` **==>** Câu điều kiện 
    - Vd: `th:if="${message != null}"` **==> message** có thể là `RedirectAttributes` hoặc `Model`
- `th:unless` **==>** phủ định của `th:if`
    - Vd: `th:if="${message != null}"` **==>** `th:unless="${message != null}"`
- `th:switch - th:case`    

```html
<td th:switch="${user.role}">
    <span th:case="admin">Quản lý hệ thống</span>
    <span th:case="editor">Duyệt bài</span>
</td>
```

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
- List box

```html
<!-- list box -->
<select name="nationality" id="nationality">
    <option
    th:each="country:${countries}"
    th:text="${country.name}"
    th:value="${country.code}"
    th:selected="${travelRequest.nationality==country.code}">China</option>
</select>

<!-- checkbox, radio button -->
<span th:each="country:${countries}">
    <input type="checkbox" name="visitedCountries"
    th:value="${country.code}"
    th:checked="${#lists.contains(travelRequest.visitedCountries, country.code)}">
    <label th:text="${country.name}" for="visitedCountries"></label><br>
</span>

<!-- hiển thị lại ô đã chọn -->
th:selected="${travelRequest.nationality==country.code}"
th:checked="${travelRequest.travelType.value==travel_type.value}">
th:checked="${#lists.contains(travelRequest.visitedCountries, country.code)}"
```

## 3. Fragment

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




