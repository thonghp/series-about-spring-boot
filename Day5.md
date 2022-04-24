# Test 

## Mục lục nội dung

  - [1. Annotation và các object thường dùng](#1-annotation-và-các-object-thường-dùng)
  - [2. Demo](#2-demo)

## 1. Annotation và các object thường dùng

- `@DataJpaTest` **==>** vô hiệu hoá `auto-configuration` và chỉ áp dụng cấu hình liên quan đến test, mặc định sử dụng h2 db thay vì db được khai báo trong tệp cấu hình
- `@AutoConfigureTestDatabase()`
    - Mặc định **==>** `replace = AutoConfigureTestDatabase.Replace.ANY`
    - `replace = AutoConfigureTestDatabase.Replace.NONE` **==>** chạy test với db thực
- `@Rollback()` **==>** chọn chế độ rollback 
    - Mặc định **==>** `true` **==>** khi thao tác xong nó sẽ rollback lại và ko lưu vào db
    - `@Rollback(false)` **==>** sẽ commit vào db luôn ko rollback lại
- `TestEntityManager` **==>** Cho phép test jpa mà ko cần phải **config/khởi tạo EntityManager** va **EntityManagerFactory** thủ công

## 2. Demo 

```java 
@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
@Rollback(false)
public class UserRepositoryTests {
    @Autowired
    private UserRepository userRepository;

    @Autowired
    private TestEntityManager entityManager;

    @Test
    public void testCreateNewUserWithOneRole() {
        // thay vì phải inject role repository để thao tác thì entityManager tìm luôn
        Role admin = entityManager.find(Role.class, 1);
        User user = new User("thong@gmail.com", "thong123", "hoàng phạm", "thông");
        user.addRole(admin);

        User savedUser = userRepository.save(user);

       // userRepository.saveAll(List.of(user1, user2));

        assertThat(savedUser.getId()).isGreaterThan(0);
    }

    @Test
    public void testGetUserById() {
        User user = userRepository.findById(1).get();

        assertThat(user).isNotNull();
    }

    @Test
    public void testCountById() {
        Integer id = 1;
        Long countById = userRepository.countById(id);

        assertThat(countById).isNotNull().isGreaterThan(0);
    }

    @Test
    public void testFindProductByName() {
        Product product = repo.findByName("iPhone 10");   

        assertThat(product.getName()).isEqualTo("iPhone 10");
    }

    @Test
    public void testListProducts() {
        List<Product> products = (List<Product>) repo.findAll();
        
        assertThat(products).size().isGreaterThan(0);
    }
}
```