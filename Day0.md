# Overview 

## 1. Spring boot là gì 

- Là 1 dự án của nền tảng Spring hoặc hệ sinh thái Spring
- Hạn chế cấu hình xml và dependency so với spring framework thuần tuý

## 2. Tính năng 

- **Tạo các ứng dụng độc lập với máy chủ nhúng (Tomcat, Jetty hoặc Undertow) ==>** không cần phải triển khai thủ công các tệp WAR.
- **Starter Dependencies** giúp đơn giản hóa việc quản lý phụ thuộc của một dự án Java
- **Automatic configuration of Spring and 3rd party libraries ==>** Spring MVC, view resolver, Spring Security, Spring Data JPA, Thymeleaf template engine
- **metrics & health checks, externalized configuration ==>** dễ dàng theo dõi ứng dụng dễ dàng

## 3. Sự khác nhau giữa Spring và Spring boot

**Spring**

- Cấu hình rất nhiều `XML`
- Quản lý version của dependency rất khủng khiếp
- Ứng dụng chạy trên external server (deploy dạng WAR)
- Thời gian thiết lập dự án lâu

**Spring boot**

- Không có cấu hình XML
- Sử dụng Starter dependency giúp đơn giản hóa đáng kể việc quản lý dependency
- Ứng dụng độc lập với máy chủ nhúng (JAR thực thi)
- Thời gian thiết lập dự án: SIÊU NHANH