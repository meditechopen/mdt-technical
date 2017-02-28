#Một số câu lệnh làm việc với MySQL

## Đăng nhập vào MySQL: mysql -u root -p

## Các câu lệnh làm việc với database:
- Xem các database có trong mysql:mysql> show databases;
- Sử dụng các database:mysql> use database_name;
- Tạo 1 database:mysql> CREATE DATABASE DB_Name;
- Sử dụng 1 database:mysql> use DB_Name;

#### Xóa database
- Xóa 1 database: DROP DATABASE DB_Name;
- Sao lưu database (quyền root): mysqldump -u root -p DB_Name > DB_Name.sql
- Sao lưu toàn bộ database (quyền root): mysqldump -u root -p --all-databases > all-databases.sql hoặc mysqldump -u root -p -A --events > all-db.sql

#### Khôi phục database với các file backup
- Khôi phục 1 database (quyền root): mysql -u root -p DB_Name < DB_Name.sql
- Khôi phục 1 database (quyền root): mysql -u root -p DB_Name < all-databases.sql
- Khôi phục toàn bộ database (quyền root): mysql -u root -p < all-db.sql


## Các thao tác với tables
- Xem các bảng có trong 1 DB: mysql> show tables;
- Hiển thị dữ liệu của 1 table: mysql> SELECT * FROM Table_name;
- Xóa 1 table: mysql> drop table Table_name;
- Sao lưu 1 table : mysqldump -u root -p DB_Name Table_name > Table_name.sql


## Các thao tác với hàng và cột
- Hiển thị các cột trong table: SHOW COLUMNS FROM Table_name;




