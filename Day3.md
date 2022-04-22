# JPA - HIBERNATE

## @Many to many 

```java
public class User {
    @ManyToMany()
    private Set<Role> roles = new HashSet<>();
}
```

`@ManyToMany(fetch = FetchType.EAGER/LAZY)`

- manyToMany và oneToMany mặc đỊnh là `LAZY`
- oneToOne và manyToOne mặc định là `EAGER`
    - `EAGER` -> Khi lấy các field của User từ csdl các role liên quan sẽ được lưu vào các User, khi giao dịch kết thúc nó vẫn được lưu trữ trong các Collection
        - Ưu điểm: luôn lấy được các đối tượng liên quan, xử lý đơn giản, thuận tiện
        - Nhược điểm: tốn nhiều thời gian và bộ nhớ khi chọn, dữ liệu lấy ra là thừa và không cần thiết.
    - `LAZY` -> Khi lấy các field của User từ csdl các role liên quan sẽ được lưu vào các User, khi giao dịch kết thúc nó sẽ không còn lưu trữ các role trong Collection nữa
        - Ưu điểm: tiết kiệm thời gian và bộ nhớ khi chọn
        - Nhược điểm: gây ra lỗi LazyInitializationException, khi bạn muốn lấy các đối tượng liên quan, bạn phải mở giao dịch lại để truy vấn