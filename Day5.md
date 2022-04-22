# Các Thuật Ngữ

## 1. Restful API

- Tiêu chuẩn dùng trong thiết kế API cho ứng dụng web service để quản lý các resource
- Ứng dụng khác nhau giao tiếp với nhau - **web, mobile**
- Không sử dụng **session và cookie**, nó sử dụng một **access token** với mỗi request

### 1.1 API 

- **Application Programming Interface ==>** các **quy tắc/cơ chế** 1 ứng dụng/thành phần **tương tác** với ứng dụng/thành phần khác 
- trả về data `JSON`, `XML`

### 1.2 REST 

- **REpresentational State Transfer ==>** chuyển đổi cấu trúc dữ liệu, để viết API
- sử dụng phương thức HTTP đơn giản để tạo cho giao tiếp giữa các máy
- Sử dụng ít băng thông hơn
- thay vì sử dụng một URL cho việc xử lý một số thông tin người dùng, REST gửi một yêu cầu HTTP như GET, POST, DELETE, vv đến một URL để xử lý dữ liệu.

### 1.3 Ưu điểm

- Giúp cho ứng dụng rõ ràng hơn
- REST URL đại diện cho resource chứ không phải hành động
- Dữ liệu được trả về với nhiều định dạng khác nhau như: xml, html, json….
- Code đơn giản và ngắn gọn
- REST chú trọng vào tài nguyên của hệ thống

### 1.4 Nguyên tắc 

