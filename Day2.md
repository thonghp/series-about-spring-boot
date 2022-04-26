# Thymeleaf

## Mục lục nội dung 

  - [1. URL](#1-url)
  - [2. Thymeleaf Standard Expression](#2-thymeleaf-standard-expression)
  
## 1. URL           

- Thuộc tính sử dụng **==>** `th:src`, `th:href`, `th:action`
- Cú pháp **==>** `"@{/...}"` 
    - Trang đầu tiên **==>** `@{/}`
    - Nối url `"@{'/users/edit/'+${user.id}}"` **==>** users/edit/1
    - absolute **==>** `@{https://frontbackend.com/tag/thymeleaf}`
    - context-relative **==>** `@{/images/logo.png}"`

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
- `th:each` **==>** Vòng lặp
    - Vd: `th:each="role: ${listRoles}"` **==>** `[[${role.name}]]` **==> listRoles** thường là **Model** 
    - Vd: `th:each="row,rowStat : ${customers}"` **==>** rowStat là biến trạng thái
- `th:custom` **==>** custom là thuộc tính tuỳ chỉnh không nằm trong html 
    - Vd: `th:userId="${user.id}"`    



