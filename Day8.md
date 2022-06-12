# Repository

  - [1. CrudRepository](#1-crudrepository)
  - [2. PagingAndSortingRepository](#2-pagingandsortingrepository)

## 1. CrudRepository

`CrudRepository<T,ID>` **==>** nhận vào **Object và Kiểu ID**

- Vd: `CrudRepository<User,Integer>`
- `findAll()` **==>** trả về danh sách ứng với T 
- `save(Entity e)` **==>** lưu entity xuống database
- `findById(int id)` **==>** truy xuất entity theo id của nó 
    - `get()` **==>** trả về đối tượng     
- `deleteById(int id)` **==>** xoá entity theo id 
- `countById(int id)` **- query creation ==>** đếm số lượng của id chỉ thị

## 2. PagingAndSortingRepository

`PagingAndSortingRepository<T,ID>`

- Vd: `PagingAndSortingRepository<User,Integer>`
- `findAll(Pageable)` **==>** phân trang
- `findAll(Sort)` **==>** sắp xếp
