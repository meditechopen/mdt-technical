# CÀI ĐẶT ZABBIX SERVER 3.0

# Mục lục

- [1. Các bước chuẩn bị](#1)
	- [1.1 LAB Topology](#11)
	- [1.2 IP Planning](#12)
- [2. Cài đặt Zabbix Server](#2)
- [3. Cài đặt Zabbix Agent](#3)
	- [3.1 Cài đặt trên Ubuntu](#31)
	- [3.2 Cài đặt trên CentOS](#32)
	- [3.3 Cài đặt trên Windows](#33)
- [4. Khác](#4)
	- [4.1 Thêm host](#41)
	- [4.2 Thêm Template cho host](#42)
	- [4.3 Thêm item cho host](#43)
	- [4.4 Thêm trigger](#44)
- [5. Tài liệu tham khảo](#5)

----------------------------------------------------

<a name="1"></a>
## 1. Các bước chuẩn bị


<a name="11"></a>
## 1.1 LAB Topology

<img src="http://i.imgur.com/oMIGpiD.png">

<a name="12"></a>
## 1.2 IP Planning

| Hostname | IP | OS |
|----------|----|----|
| Zabbix-SRV | 192.168.100.40 | Ubuntu 14.04 |
| U14 | 192.168.100.42 | Ubuntu 14.04 |
| CentOS 6 | 192.168.100.31 | CentOS 6 |
| Windows | 192.168.100.144 | Windows SRV 2012 |


<a name="2"></a>
## 2. Cài đặt Zabbix Server

### Thực hiện các bước sau trên node Zabbix Server


- Bước 1: Tải gói cài đặt Zabbix

```sh 
wget http://repo.zabbix.com/zabbix/3.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_3.0-1+trusty_all.deb

dpkg -i zabbix-release_3.0-1+trusty_all.deb

apt-get update
```



Bước 2: Tải gói cài đặt Mysql

```sh
apt-get install zabbix-server-mysql zabbix-frontend-php
```

- Bước 3: Thay đổi mật khẩu cho user Mysql

```sh
mysql_secure_installation
```

Bước 4:  Đăng nhập vào mysql, tạo user và phân quyền

```
mysql -uroot -p

create database zabbix character set utf8 collate utf8_bin;

grant all privileges on zabbix.* to zabbix@localhost identified by 'MDT2017';

quit;
```

Bước 5: Import

```sh
zcat /usr/share/doc/zabbix-server-mysql/create.sql.gz | mysql -uzabbix -p zabbix
```


Bước 6: Sửa file cấu hình như sau

```sh
vi /etc/zabbix/zabbix_server.conf

DBHost=localhost
DBName=zabbix
DBUser=zabbix
DBPassword=MDT2017
```

Bước 7: Khởi động lại dịch vụ và cho dịch vụ khởi động cùng hệ thống

```sh
service zabbix-server start
update-rc.d zabbix-server enable
```

- Bước 8: Sửa file timezone

```sh
vi /etc/zabbix/apache.conf

php_value max_execution_time 300
php_value memory_limit 128M
php_value post_max_size 16M
php_value upload_max_filesize 2M
php_value max_input_time 300
php_value always_populate_raw_post_data -1
php_value date.timezone Asia/Ho_Chi_Minh
```

- Bước 9: Khởi động lại dịch vụ apache2

```sh
service apache2 restart
```

- Bước 10: Truy cập vào Web 

http://<IP-ZABBIX-SRV>/zabbix


<a name="3"></a>
## 3. Cài đặt Zabbix Agent

### Thực hiện các bước sau trên node client

<a name="31"></a>
### 3.1 Cài đặt trên Ubuntu

- Bước 1: Tải các gói cài đặt, tùy vào phiên bản của Ubuntu lựa chọn một trong các gói cài đặt thích hợp


#### Trên U16.04

wget http://repo.zabbix.com/zabbix/3.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_3.0-1+xenial_all.deb
dpkg -i zabbix-release_3.0-1+xenial_all.deb
apt-get update


#### Trên U14.04
wget http://repo.zabbix.com/zabbix/3.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_3.0-1+trusty_all.deb
dpkg -i zabbix-release_3.0-1+trusty_all.deb
sudo apt-get update


#### Trên U12
wget http://repo.zabbix.com/zabbix/2.2/ubuntu/pool/main/z/zabbix-release/zabbix-release_2.2-1+precise_all.deb
dpkg -i zabbix-release_2.2-1+precise_all.deb
apt-get update


#### Ở đây do tôi dùng U14, nên chỉ tải gói của U14

- Bước 2: Tải gói Zabbix-agent

```sh
apt-get install zabbix-agent -y
```


- Bước 3: Sửa file cấu hình

```sh
vi /etc/zabbix/zabbix_agentd.conf

Server=Zabbix-SRV-IP

ServerActive=Zabbix-SRV-IP

Hostname=
```

- Bước 4: Khởi động lại dịch vụ & khởi động cùng hệ thống

```sh
/etc/init.d/zabbix-agent restart

update-rc.d zabbix-agent

update-rc.d zabbix-agent defaults
```


<a name="32"></a>
## 3.2 Cài đặt trên CentOS


- Bước 1: Tải các gói cài đặt, tùy vào phiên bản của CentOS lựa chọn một trong các gói cài đặt thích hợp


#### Trên CentOS 7
rpm -Uvh http://repo.zabbix.com/zabbix/3.0/rhel/7/x86_64/zabbix-release-3.0-1.el7.noarch.rpm

#### Trên CentOS 6
rpm -Uvh http://repo.zabbix.com/zabbix/3.0/rhel/6/x86_64/zabbix-release-3.0-1.el6.noarch.rpm

#### Ở đây do tôi dùng CentOS 6, nên chỉ tải gói của CentOS 6 

- Bước 2: Cài đặt Zabbix-agent

```sh
yum install zabbix zabbix-agent -y
```

- Bước 3: Sửa file cấu hình

```sh
vi /etc/zabbix/zabbix_agentd.conf

Server=Zabbix-SRV-IP

ServerActive=Zabbix-SRV-IP

Hostname=
```

- Bước 4: Khởi động lại dịch vụ & khởi động cùng hệ thống

```sh
/etc/init.d/zabbix-agent restart
Hoặc
systemctl start zabbix-agent

chkconfig zabbix-agent on
```


- Bước 5: Trong 1 số trường hợp cần mở rule iptables để Zabbix-server có thể liên lạc được với agent

```sh
vi /etc/sysconfig/iptables


-A INPUT -p tcp -m state --state NEW -m tcp --dport 10050 -j ACCEPT
```


<a name="33"></a>
## 3.3 Cài đặt trên Windows


- Bước 1: Tải gói cài đặt Zabbix-agent tại đây

```sh
http://www.zabbix.com/download
```

<a name="4"></a>
## 4. Khác




















