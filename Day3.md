# JPA 

## Mục lục nội dung 

  - [1. Annotation](#1-annotation)
  - [2. Many to many](#2-many-to-many)

## 1. Annotation

- `@Entity` **==>** chỉ định một class là entity
- `@Table` **==>** map class đến table 
- `@Id` **==>** chỉ định trường khoá chính
- `@GeneratedValue` **==>** 
    - `strategy = GenerationType.IDENTITY` **==>** chỉ định trường tự tăng (**thường là khoá chính**)
- `@Column` **==>** chỉ định thêm thuộc tính cho cột, không chỉ định `@Column` thì trường đó hiển nhiên nhận giá trị default 
    -  `name = "first_name", length = 40, nullable = false, unique = true` **==>** `varchar, 40, not null, unique` 
    - `@Nationalized` **==>** chỉ định kiểu `nvarchar`
- `@ManyToMany()` **==>** Kết bảng nhiều - nhiều
    - `fetch = FetchType.EAGER/LAZY` **==>** Mặc định là **LAZY**
        - `ManyToMany`, `OneToMany` **==>** Mặc định là **LAZY**
        - `OneToOne` và `ManyToOne` **==>** Mặc định là **EAGER**
- `@JoinTable` **==>** join 2 bảng
    - `@JoinColumn` **==>** chỉ định tên cột join
- `@Transient` **==>** thuộc tính này sẽ không lưu vào db
    - Nếu trường đó **implement serializable** thì dùng keyword transient mới bỏ qua không lưu vào db

## 2. Many to many

- Override `hashCode()` and `equals()` **==>** thường là id 

```java
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (o == null || getClass() != o.getClass()) return false;
        Role role = (Role) o;
        return id.equals(role.id);
    }

@Override
public int hashCode() {
    return Objects.hash(id);
}
```

- 1 user có nhiều role 1 và nhiều user cùng 1 role **==>** `@ManyToMany` để ở entity user 
    - `name` của `JoinTable` **==>** tên của bảng trung gian
    - `name` của `JoinColumn` **==>** tên của 2 trường map với id của 2 bảng 

```java
@ManyToMany()
@JoinTable(name = "user_role",
        joinColumns = @JoinColumn(name = "user_id"),
        inverseJoinColumns = @JoinColumn(name = "role_id"))
private Set<Role> roles = new HashSet<>();
```

> - **EAGER ==>** Khi lấy các field của User từ db thì các role liên quan sẽ được lấy theo các User, khi **giao dịch kết thúc** nó vẫn được **lưu trữ** trong các Collection
>   - **Ưu điểm ==>** luôn lấy được các đối tượng liên quan, xử lý đơn giản, thuận tiện
>   - **Nhược điểm ==>** tốn nhiều thời gian và bộ nhớ khi chọn, dữ liệu lấy ra là thừa và không cần thiết.
> - **LAZY ==>** Tương tự như EAGER nhưng khi giao dịch kết thúc thì nó sẽ **không còn lưu trữ** các role trong Collection 
>   - **Ưu điểm ==>** tiết kiệm thời gian và bộ nhớ khi chọn
>   - **Nhược điểm ==>** gây ra lỗi **LazyInitializationException**, khi bạn muốn lấy lại các đối tượng liên quan, bạn phải mở giao dịch lại để truy vấn


