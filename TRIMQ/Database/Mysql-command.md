#Một số câu lệnh làm việc với MySQL

# Mục lục
- [1. Các câu lệnh làm việc với Database](#1)
- [2. Các câu lệnh làm việc với table trong Database](#2)
- [3. Các câu lệnh làm việc với hàng và cột](#3)

## Đăng nhập vào MySQL: mysql -u root -p


<a name="1"></a>
## 1. Các câu lệnh làm việc với database:
- Xem các database có trong mysql:
```sh
mysql> show databases;
```
- Sử dụng các database:
```sh
mysql> use database_name;
```
- Tạo 1 database:
```sh
mysql> CREATE DATABASE DB_Name;
```
- Sử dụng 1 database:
```sh
mysql> use DB_Name;
```

- Kiểm tra size của database

```sh
SELECT table_schema "Database Name", SUM(data_length+index_length)/1024/1024
"Database Size (MB)"  FROM information_schema.TABLES GROUP BY table_schema;
```

#### Xóa database
- Xóa 1 database: DROP DATABASE DB_Name;
- Sao lưu database (quyền root): 
```sh
mysqldump -u root -p DB_Name > DB_Name.sql
```
- Sao lưu toàn bộ database (quyền root): 
```sh
mysqldump -u root -p --all-databases > all-databases.sql hoặc mysqldump -u root -p -A --events > all-db.sql
```

#### Khôi phục database với các file backup
- Khôi phục 1 database (quyền root): mysql -u root -p DB_Name < DB_Name.sql
- Khôi phục 1 database (quyền root): mysql -u root -p DB_Name < all-databases.sql
- Khôi phục toàn bộ database (quyền root): mysql -u root -p < all-db.sql


<a name="2"></a>
## 2. Các thao tác với tables
- Xem các bảng có trong 1 DB: mysql> show tables;
- Hiển thị dữ liệu của 1 table: mysql> SELECT * FROM Table_name;
- Xóa 1 table: mysql> drop table Table_name;
- Sao lưu 1 table : mysqldump -u root -p DB_Name Table_name > Table_name.sql


<a name="3"></a>
## 3. Các thao tác với hàng và cột
- Hiển thị các cột trong table: SHOW COLUMNS FROM Table_name;






