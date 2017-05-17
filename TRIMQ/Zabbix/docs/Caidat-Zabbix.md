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
- [4. Thao tác với các host giám sát](#4)
	- [4.1 Thêm host](#41)
	- [4.2 Thêm Template cho host](#42)
	- [4.3 Thêm item cho host](#43)
	- [4.4 Thêm trigger](#44)
- [5. Khác](#5)
	- [5.1 Chỉnh sửa template](#51)
	- [5.2 Sử dụng template ICMP](#52)
- [6. Tài liệu tham khảo](#6)

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

```sh
wget http://repo.zabbix.com/zabbix/3.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_3.0-1+xenial_all.deb
dpkg -i zabbix-release_3.0-1+xenial_all.deb
apt-get update
```

#### Trên U14.04

```sh
#### Trên U14.04
wget http://repo.zabbix.com/zabbix/3.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_3.0-1+trusty_all.deb
dpkg -i zabbix-release_3.0-1+trusty_all.deb
sudo apt-get update
```

#### Trên U12

```sh
wget http://repo.zabbix.com/zabbix/2.2/ubuntu/pool/main/z/zabbix-release/zabbix-release_2.2-1+precise_all.deb
dpkg -i zabbix-release_2.2-1+precise_all.deb
apt-get update
```

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

Một số thao tác với Zabbix

<a name="41"></a>
### 4.1 Thêm host

Để thêm 1 host mới trên dashboard Zabbix, làm theo các bước sau

- Bước 1: Thực hiện add host theo các bước sau

<img src="http://i.imgur.com/lNofIR2.png">

- Bước 2: Nhập các thông số sau

<img src="http://i.imgur.com/lYH2TAW.png">

<ul>
<li>Mục 1: Nhập vào tên host trên dashboard</li>
<li>Mục 2: Thêm host giám sát vào các host group có sẵn mà</li>
<li>Mục 3: Thêm host giám sát vào 1 group mới</li>
<li>Mục 4: Nhập địa chỉ IP của host giám sát</li>
</ul>

Sau khi hoàn thành các bước trên chọn `Add`

- Bước 3: Kiểm tra trên Dashboard

<img src="http://i.imgur.com/mfl2qAV.png">


<a name="42"></a>
### 4.2 Thêm Template cho host

Trong Zabbix đã có sẵn các mẫu để giám sát các thông số tài nguyên hệ thống như giám sát với máy chủ có OS Linux, Windows...Khi thêm các templates này vào host giám sát, các templates sẽ có sẵn các item, trigger, graph..

- Bước 1: Làm theo các bước sau để thêm template vào host giám sát

<img src="http://i.imgur.com/roBGd4Q.png">

- Bước 2: Chọn template, do ở đây host giám sát của tôi sử dụng Ubuntu14 nên tôi sẽ thêm template `OS Linux`

<img src="http://i.imgur.com/WKRvjA8.png">

Sau đó chọn nút `Add` và `Update` trạng thái

- Bước 3: Kiểm tra trên Dashboard

<img src="http://i.imgur.com/ec5DOlP.png">

Như vậy là tôi đã thêm template cho Host giám sát.

<a name="43"></a>
### 4.3 Thêm item cho host

Item của Zabbix là những key có sẵn (hoặc có thể tự viết) để thu thập dữ liệu từ các host được giám sát. Cách thêm như sau:

- Bước 1: Tạo 1 item

<img src="http://i.imgur.com/mskUtob.png">

- Bước 2: Điền các thông tin item

<img src="http://i.imgur.com/J27eobB.png">

<ul> 
<li>Name: là tên được đặt cho item</li>
<li>Type: Loại item này thuộc Zabbix-agent</li>
<li>Kye: Đây là các key được quy định sẵn trong zabbix-agent, chọn key phù hợp</li>
<li>Host interface: Là địa chỉ IP của host giám sát với port 10050</li>
<li>Type of information: Đây là loại dữ liệu item trả về</li>
<li>Data type: Loại dữ liệu</li>
<li>Units: Đơn vị của dữ liệu, đây là thông số cực kì quan trọng. Cần xác định đúng đơn vị của dữ liệu trả về để giám sát đúng</li>
</li>Applications: Tầng ứng dụng, nơi mà item sẽ nằm trong đó. Ví dụ, RAM used sẽ nằm trong ứng dụng là Memory</li>
</ul>

Sau khi điền xong các thông số cơ bản trên, chọn `Add`

#### Ví dụ:

- Tôi sẽ tạo 1 item giám sát phần trăm RAM đã sử dụng

<img src="http://i.imgur.com/C1IQgtp.png">

- Kiểm tra trên Dashboard

<img src="http://i.imgur.com/qOiNucq.png">

Như vậy là tôi đã thêm thành công 1 item

<a name="44"></a>
### 4.4 Thêm trigger

Trigger là các ngưỡng cảnh báo được đặt ra dựa theo thông số mà các item thu được từ host giám sát. Để add các trigger làm theo các bước sau

- Bước 1: Thực hiện thêm trigger

<img src="http://i.imgur.com/undefined.png">

- Bước 2: Điền các thông số sau

<img src="http://i.imgur.com/2t5HiVV.png">

<ul>
<li>Name: tên của trigger</li>
<li>Expression: Là các biểu thức tính toán để đưa ra cảnh báo</li>
<li>Severity: Là các mức độ nghiêm trọng của cảnh báo</li>
</ul>

- Bước 2.1: Sử dụng các `Expression`

Khi add các Expression sẽ làm như sau:

<img src="http://i.imgur.com/mFm3MpQ.png">

<ul>
<li>Item: Là các item thu thập dữ liệu, chọn 1 item muốn cảnh báo</li>
<li>Function: Là biểu thức tính giá trị của dữ liệu thu được. Ví dụ, lần cuối cùng lớn hơn giá trị N, giá trị trung bình của dữ liệu...</li>
<li>Last of (T): Khoảng thời gian check cuối cùng</li>
<li>N: Là giá trị xác định ngưỡng, nếu quá ngưỡng giá trị này sẽ cảnh báo</li>
</ul>

Sau khi thực hiện xong các bước, chọn `Insert`

- Bước 3: Kiểm tra trên Dashboard

<img src="http://i.imgur.com/2t5HiVV.png">

- Bước 4: Chọn `Add` và kiểm tra

<img src="http://i.imgur.com/5nf9ejA.png">

Như vậy là tôi đã tạo trigger cho giá trị cần giám sát




































