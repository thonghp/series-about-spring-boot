# Thymeleaf

## Mục lục nội dung 

  - [1. URL](#1-url)
  - [2. Form](#2-form)
- [Thymeleaf Standard Expression](#thymeleaf-standard-expression)

## 1. URL           

- Thuộc tính sử dụng **==>** `th:src`, `th:href`, `th:action`
- Cú pháp **==>** `"@{/...}"`
    - Nối url `"@{'/users/edit/'+${user.id}}"` **==>** users/edit/1

## 2. Form 

- Thuộc tính sử dụng **==>** `th:action`, `th:object`, `th:field`, `th:block`, `th:each`, `th:text`, `th:value`
- Cú pháp
    - `th:action` **==>** điều hướng tới server xử lý submit form, tương đương với `th:attr="action="`
        - Vd: `th:action="@{/users/save}"`
    - `th:object` **==>** map tới entity, tương đương với `session.entity`
        - Vd: `th:object="${user}"`
    - `th:field` **==>** thuộc tính của object map với entity, tương đương với `session.entity.properties`, xem như là **id**
        - Vd: `th:field="*{email}"`
    - `th:block` **==>** tạo ra 1 vùng chứa để thực thi và sau khi xong sẽ tự mất, thường đi với `th:each`
        - Vd: `<th:block th:each="role: ${listRoles}"></th:block>`
    - `th:each`
        - Vd: `th:each="role: ${listRoles}"` **==>** `role.name`,...
        - Vd: `th:each="row,rowStat : ${customers}"` **==>** rowStat là biến trạng thái
    - `th:text` **==>** Đổ data dưới dạng text tương đương với `[[${role.description}]]` - **Nội tuyến**
    - `th:value` **==>** lưu giá trị    
    - 

# Thymeleaf Standard Expression

Basic [ở đây](https://loda.me/articles/sb9-gii-thch-cch-thymeleaf-vn-hnh-expression-demo-full)

- Trang hiện tại => `@{/}` 
- Duyệt for-each  => `th:each="user: ${listUsers}`
    - truy cập thuộc tính => `[[${user.id}]]`
- `th:if` => Câu điều kiện   
- `th:form` => trong form chỉ có 1 thuộc tính, chứa tham chiếu đến command object
- `th:field` => được sử dụng để liên kết với getter của một thuộc tính trong lớp bean
