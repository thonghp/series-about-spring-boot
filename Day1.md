# Spring

## Mục lục nội dung 

  - [1. Các kiểu tổ chức project](#1-các-kiểu-tổ-chức-project)
  - [2. Quy trình hoạt động](#2-quy-trình-hoạt-động)
  - [3. Cơ chế IoC](#3-cơ-chế-ioc)
  - [4. Các Annotation](#4-các-annotation)
    - [4.1 @SpringBootApplication - @EntityScan - @Component](#41-springbootapplication---entityscan---component)
    - [4.2 @Repository - @Service - @Controller](#42-repository---service---controller)
    - [4.3 @Autowired - @Primary - @Qualifier](#43-autowired---primary---qualifier)
    - [4.4 @PostMapping - @GetMapping - @RequestMapping](#44-postmapping---getmapping---requestmapping)
    - [4.5 @RestController - @PathVariable](#45-restcontroller---pathvariable)
    - [4.6 @Query - @Param](#46-query---param)
    - [4.7 @Configuration - @Bean](#47-configuration---bean)
    - [4.8 @Transactional - @Modifying](#48-transactional---modifying)
  - [5. Model](#5-model)
  - [6. RedirectAttributes](#6-redirectattributes)
  - [7. Spring security](#7-spring-security)
    - [7.1 @EnableWebSecurity và WebSecurityConfigurerAdapter](#71-enablewebsecurity-và-websecurityconfigureradapter)
    - [7.2 BCrypt](#72-bcrypt)

## 1. Các kiểu tổ chức project

Có 2 kiểu tổ chức thường dùng là package by feature và layer

- **package by feature** khuyên nên sử dụng 

    ![alt img](/assets/package-by-feature.jpg)

- **layer**

    ![alt img](/assets/layer.jpg)

## 2. Quy trình hoạt động

Request gửi **==>** Controller sẽ gọi **==>** Service trả về kết quả **==>** Controller sẽ demand **==>** Model trả về data **==>** Controller chuyển tới **==>** View

## 3. Cơ chế IoC 

- Spring IoC - **Inversion of Control** là cốt lõi của spring framework
  - Tạo object, cấu hình, lắp ráp dependency
  - Container sử dụng DI để quản lý 
- **Dependency là Bean** và Bean ở trong **ApplicationContext** tất cả đều là **singleton**  
- Có 2 loại IoC container là `BeanFactory` và `ApplicationContext` - **mở rộng các tính năng của BeanFactory**, là nơi lưu trữ **Bean**, bao gồm các tính năng chính sau
  - Tạo và quản lý object
  - Định cấu hình, quản lý dependencies

## 4. Các Annotation

Khi chạy ứng dụng sẽ tìm `@Component` và `@Configuration` chạy trước

### 4.1 @SpringBootApplication - @EntityScan - @Component

- `@SpringBootApplication` **==>** Sử dụng cho **main class**
  - Đóng gói `@EnableAutoConfiguration` - **Cấu hình tự động**, `@ComponentScan` - **Quét component** và `@Configuration` - **Cấu hình bổ sung** bên trong 
    - Chạy hàm **SpringApplication.run(...)** **==>** tìm Bean đưa vào Context
- `@EntityScan` **==>** Chỉ định nơi entity muốn quét
  - `@Entity` không nằm cùng **package** hay **sub-package** của **main application** 
  - Sử dụng `@EntityScan` sẽ tắt tính năng `@EnableAutoConfiguration`
- `@Component` **==>** đánh dấu là Bean

> Xem thêm [ở đây](https://loda.me/articles/sb1-huong-dn-component-va-autowired)

### 4.2 @Repository - @Service - @Controller

Cả 3 annotation trên cũng là `@Component`

- `@Controller` **==>** chỉ dùng cho class **==>** tầng giao tiếp với view và xử lý các Request từ view tới server 
- `@Service` **==>** chỉ dùng cho class **==>** tầng service phục vụ xử lý logic, nghiệp vụ
- `@Repository` **==>** phục vụ truy suất data trả về cho tầng service và chuyển đổi **database exception** thành exception không được kiểm tra **Spring-based**

`view ==> controller ==> service ==> repository ==> entity`

> Xem thêm [ở đây](https://loda.me/articles/sb4-component-vs-service-vs-repository)

### 4.3 @Autowired - @Primary - @Qualifier

- `@Autowired` **==>** Tự động inject 1 instance của thuộc tính khi Bean khởi tạo
    - inject thông qua **constructor > setter > java reflection**  

> Xem thêm [ở đây](https://loda.me/articles/sb2-autowired-primary-qualifier)

### 4.4 @PostMapping - @GetMapping - @RequestMapping

- Create **==>** `POST`, Read **==>** `GET`, Update **==>** `PUT/PATCH`, Delete **==>** `DELETE`
- `@GetMapping` **==>** map đến http get 
- `@PostMapping` **==>** map đến http post 

> Xem thêm [ở đây](https://loda.me/articles/sb10-requestmapping-postmapping-modelattribute-requestparam-web-to-do-voi-thymeleaf)

### 4.5 @RestController - @PathVariable

- `@RestController` **==>** trả về data dạng json **==>** object sẽ được chuyển thành **JSON**
  - Kết hợp `@Controller` + `@ResponseBody` giúp xây dựng Restful API dễ đàng hơn
- `@PathVariable` **==>** Lấy giá trị tham số là 1 thành phần trong url `{}` 

```java
@GetMapping("/users/{id}/enabled/{status}")
public String updateUserEnabledStatus(@PathVariable(name = "id") Integer id, @PathVariable(name = "status") boolean enabled)
```

### 4.6 @Query - @Param

- `@Query` **==>** thực hiện câu truy vấn, có 2 kiểu là tham số và index

```java
// theo kiểu tham số
@Query("SELECT u FROM User u WHERE u.email = :email")
public User getUserByEmail(@Param("email") String email);

// theo kiểu index
@Query("UPDATE User u SET u.enabled = ?2 WHERE u.id = ?1")
public void updateEnabledStatus(Integer id, boolean enabled);
```

- `@Param` **==>** liên kết với tham số `:` của `@Query`, hoặc thuộc tính gửi từ client về server

```java
@PostMapping("/users/check_email")
public String checkDuplicateEmail(@Param("id") Integer id, @Param("email") String email) {
    return userService.isEmailUnique(id, email) ? "OK" : "Duplicated";
}
```

### 4.7 @Configuration - @Bean 

- `@Configuration` cũng là một `@Component` nhưng `@Component` không thể hoạt động giống `@Configuration`
    - Khai báo 1 hoặc nhiều `@Bean`
- `@Bean` **Thường được khai báo** trong lớp `@Configuration`
    - Khởi tạo instance sau đó đưa bean vào trong `Context`
    - Có thể tham chiếu đến các bean khác trong cùng 1 lớp

> Xem thêm [ở đây](https://loda.me/articles/sb6-configuration-va-bean)

### 4.8 @Transactional - @Modifying

- `@Modifying` **==>** dùng thực thi truy vấn DML (**INSERT, UPDATE, DELETE,...**)
- `@Transactional` **==>** Sử dụng khi thao tác DML 
  - Khai báo public

## 5. Model

- `Model` lưu trữ thông tin dưới dạng **key-value** và ví như `Context` của thymeleaf. 
- Đây là 1 object được **đính kèm** trong mỗi response
- Chứa thông tin trả về và Template Engine sẽ lấy thông tin này đưa vào html

> xem thêm [ở đây](https://loda.me/articles/sb8-tao-web-helloworld-voi-controller-thymeleaf)

## 6. RedirectAttributes 

- `addFlashAttribute()` lưu trữ thuộc tính trong flashmap (có thể lưu trữ được nhiều loại object)
- tồn tại nội bộ trong phiên người dùng và bị xóa khi chuyển hướng sang chổ khác

## 7. Spring security

### 7.1 @EnableWebSecurity và WebSecurityConfigurerAdapter

- `@EnableWebSecurity` Sử dụng để kích hoạt spring security và thường đi chung với `WebSecurityConfigurerAdapter`
- `WebSecurityConfigurerAdapter` là một interface tiện ích giúp cài đặt thông tin    dễ dàng hơn

### 7.2 BCrypt

- Là 1 provider của spring security
- dựa trên blowfish và crypt function trong funix
- mã hoá ra chuỗi dài 60 ký tự
- Nó có thể tăng lên khi máy tính chạy nhanh hơn
- chống lại rainbow table attack

BCryptPasswordEncoder

encode => Encode the raw password.
matches => xác thực xem raw password và encode có giống nhau không


