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
- `@JoinTable` **==>** join 2 bảng
    - `@JoinColumn` **==>** chỉ định tên cột join
- `@Transient` **==>** thuộc tính này sẽ không lưu vào db
    - Nếu trường đó **implement serializable** thì dùng keyword transient mới bỏ qua không lưu vào db

## 2. Many to many

- Override `hashCode()` and `equals()` **==>** thường là id 
    - Mục đích sử dụng khi so sánh 2 đối tượng thì nó so sánh tham chiếu **==>** phải ghi đè **==>** chọn id **==>** để hạn chế nếu ta chỉ override lại instance thì nó sẽ equals và hash tất cả thuộc tính

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

## 3. Lazy và Eager    
    
- Thường là select và find thể hiện rõ 2 tính chất này
- EAGER sẽ lấy tất cả mọi thứ 1 lần **==> chậm**
    - `OneToOne` và `ManyToOne` **==>** Mặc định là **EAGER**
    - Vd: khi query khoá học thì mặc định nó query luôn object sinh viên khi kết thúc thì object vẫn còn 
- LAZY sẽ lấy khi có request **==> nhanh** nhưng dễ xảy ra lỗi **LazyInitializationException**
    - `ManyToMany` và `OneToMany` **==>** Mặc định là **LAZY**
    - Vd: khi query khoá học thì nó không lấy object sinh viên liên quan nhưng nó vẫn nằm trong transaction khi kết thúc thì object sinh viên sẽ xoá theo


