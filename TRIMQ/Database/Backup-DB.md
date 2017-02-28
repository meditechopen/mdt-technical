# Backup database

Tài liệu sẽ được cập nhật thêm...

# Mục lục:
- [1. Cài đặt Wordpress](#1)
- [2. Backup database của Wordpress](#2)
- [3. Backup các tables trong database](#3)
- [4. Tài liệu tham khảo](#4)

----------------------------------

<a name="1"></a>
## 1. Cài đặt Wordpress

### Cài đặt Apache
- Update các gói phần mềm

```sh
apt-get update
```

- Tải gói Apache

```sh
apt-get install apache2 -y
```

### Cài đặt MySQL
- Tải các gói cài đặt MySQL

```sh
apt-get install mysql-server php5-mysql -y
```

- Tạo cấu trúc cho cơ sở dữ liệu

```sh
mysql_install_db
```

### Cài đặt PHP
- Tải các gói cài đặt PHP

```sh
apt-get install php5 libapache2-mod-php5 php5-mcrypt -y 
```

- Sửa thư mục `/etc/apache2/mods-enabled/dir.conf` như sau:

```sh
<IfModule mod_dir.c>
    DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
</IfModule>
```

- Khởi động lại dịch vụ Apache

```sh
service apache2 restart
```

- Kiểm tra lại bằng cách truy cập vào địa chỉ IP của Server trên Web

### Tạo database cho Wordpress
- Đăng nhập vào database với quyền root

```sh
mysql -u root -p
```

- Tạo database có tên wordpress:

```sh
CREATE DATABASE wordpress;
```

- Tạo user cho database

```sh
CREATE USER quoctrimai@localhost IDENTIFIED BY 'Welcome123';
```

- Cấp quyền cho user vừa tạo truy cập vào database `wordpress`

```sh
GRANT ALL PRIVILEGES ON wordpress.* TO quoctrimai@localhost;
```

- Khai báo thêm

```sh
FLUSH PRIVILEGES;
```

- Thoát khỏi mysql

```sh
exit;
```

### Cài đặt Wordpress
- Tải gói cài đặt wordpress

```sh
wget http://wordpress.org/latest.tar.gz
```

- Giải nén gói cài đặt

```sh
tar xzvf latest.tar.gz
```

- Tải 1 số gói cần thiết

```sh
apt-get install php5-gd libssh2-php -y
```

- Chỉnh sửa cấu hình wordpress trong file `wp-config.php`

```sh
// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define('DB_NAME', 'wordpress');

/** MySQL database username */
define('DB_USER', 'quoctrimai');

/** MySQL database password */
define('DB_PASSWORD', 'Welcome123');
```

- Sử dụng câu lệnh rsync chuyển các file vào thư mục `/var/www/html/` để truy cập được vào web

```sh
rsync -avP ~/wordpress/ /var/www/html/
```

- Vào địa chỉ IP của server để vào giao diện web

<a name="2"></a>
## 2. Backup database của Wordpress
Khi xóa DATABASE của wordpress đi, sẽ không truy cập được WordPress được nữa. Phải thực hiện backup trước khi xóa
<img src="http://prntscr.com/ec9z4a">

- Sử dụng câu lệnh `mysqldump` để backup dữ liệu database

```sh
mysqldump wordpress > backupsql.sql
```

- Để restore lại ta cần tạo 1 database mới rồi restore file lại

```sh
mysql -u root -p

CREATE DATABASE wordpress;

exit;
```

- Restore file backup vào database

```sh
mysql wordpress < backupsql.sql
```

- Có thể chuyển file backup sang 1 máy chủ khác trong trường hợp máy chủ gặp sự cố

```sh
scp backupmysql.sql root@192.168.100.35:/home/backupdb
```

<a name="3"></a>
## 3. Backup các tables trong database
- Sử dụng câu lệnh `mysqldump` để backup các tables trong database

- Đăng nhập vào database

```sh
mysql -u root -p
```

- Xem các database

```sh
show databases;

+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| wordpress          |
+--------------------+
4 rows in set (0.00 sec)
```

- Xem các tables của 1 database

==> Chọn 1 database để sử dụng

```sh
use DB_name;
```

==> Xem các tables của database đó

```sh
show tables;
```

- Tạo backup cho 1 bảng trong database:

```sh
mysqldump -u -p DB_NAME TABLE_NAME > TABLE_NAME.sql
```

- Tạo backup cho nhiều bảng trong database:

```sh
mysqldump -u -p DB_NAME TABLE_NAME1 TABLE_NAME2 > TABLE_NAME.sql
```

- Restore lại tables trong trường hợp tables bị mất:

```sh
mysql -u -p DB_NAME < TABLE_NAME.sql
```

<a name="4"></a>
## 4. Tài liệu tham khảo:
- (1) http://stackoverflow.com/questions/9696249/restoring-a-mysql-table-back-to-the-database
- (2) http://hocvps.com/cac-cau-lenh-mysql-can-phai-biet/













