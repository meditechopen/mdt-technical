# Công cụ Grafana

# Mục lục

- [1. Giới thiệu về Grafana](#1)
	- [1.1 Một số thuật ngữ dùng trong Grafana](#11)
- [2. Cài đặt grafana](#2)
	- [2.1 Mô hình cài đặt và IP Planning](#21)
	- [2.2 Cài đặt trên Ubuntu](#22)
	- [2.3 Cài đặt trên Centos](#23)
- [3. Hướng dẫn sử dụng dashboard grafana](#3)
	- [3.1 Thêm Plugin Zabbix vào Grafana](#31)
	- [3.2 Thêm Data Sources](#32)
- [4. Tạo 1 dashboard cơ bản](#4)
	- [4.1 Singlestat](#41)
	- [4.2 Graph](#42)
	- [4.3 Pie Chart](#43)
- [5. Khác](#5)
- [6. Tài liệu tham khảo](#6)

------------------------------------------------------


<a name="1"></a>
## 1. Giới thiệu về Grafana
Là công cụ mã nguồn mở phân tích số liệu một cách trực quan. Thường được sử dụng để hiển thị dữ liệu chuỗi của các cơ sở hạ tầng và phân tích ứng dụng. Grafana là 1 bảng rất đẹp hiển thị các chỉ số Graphite khác nhau thông qua trình duyệt web 



<a name="11"></a>
### 1.1 Một số thuật ngữ dùng trong Grafana

| Thuật ngữ | Ý nghĩa |
|-----------|---------|
| Data sources | Grafana hỗ trợ nhiều backend lưu trữ khác nhau, các data sources được hỗ trợ như  Graphite, InfluxDB, OpenTSDB, Prometheus, Elasticsearch, CloudWatch.|
| Organization | Grafana hỗ trợ nhiều tổ chức để hỗ trợ nhiều mô hình triển khai khác nhau, bao gồm việc sử dụng một trường hợp Grafana đơn để cung cấp dịch vụ cho nhiều Tổ chức có thể không đáng tin cậy.|
| User | User là một tài khoản được đặt tên ở Grafana. User có thể thuộc một hoặc nhiều Organization và có thể được phân cấp các mức quyền ưu tiên khác nhau thông qua vai trò |
| Row | Một hàng là một bộ phận chia lôgíc hợp lý trong một Dashboard và được sử dụng để nhóm Panel với nhau |
| Panel | Là các khối block trong Grafana để hiển thị sử dụng Editor Query, như Graph, Singlestat, Dashlist, Table, và Text |
| Query Editor | Query Editor cho khả năng truy xuất đến data sources |
| Dashboard | Là 1 bảng điều khiển gồm nhiều Panel sắp xếp lại với nhau |

<a name="2"></a>
## 2. Cài đặt grafana

- Cài đặt Grafana trên các distro linux khác nhau. Phiên bản ổn định mới nhất đang là Grafana 4.2.0 (x86-64 deb)

- Chọn 1 trong 2 distro sau đây để thực hiện cài đặt. Ở đây tôi dùng OS là Ubuntu


<a name="21"></a>

### 2.1 Mô hình cài đặt và IP Planning

- <b>MÔ HÌNH CÀI ĐẶT:</b>

<img src="http://i.imgur.com/IHCEoCl.png">


- <b>IP Planning</b>

| Host name | IP | OS |
|-----------|----|----|
| Grafana | 192.168.100.42 | Ubuntu 14 |
| Zabbix | 192.168.100.40 | Ubuntu 14 |



<a name="22"></a>
### 2.2 Cài đặt trên Ubuntu

#### Thực hiện trên Ubuntu Server 14

- Bước 1: Tải gói cài đặt Grafana và giải nén

```sh
wget https://s3-us-west-2.amazonaws.com/grafana-releases/release/grafana_4.2.0_amd64.deb
apt-get install -y adduser libfontconfig
dpkg -i grafana_4.2.0_amd64.deb
```

- Bước 2: Thêm dòng sau vào file `/etc/apt/sources.list`

```sh
echo "deb https://packagecloud.io/grafana/stable/debian/ jessie main" >> /etc/apt/sources.list
```

- Bước 3: Add Package cloud key

```sh
curl https://packagecloud.io/gpg.key | sudo apt-key add -
```

- Bước 4: Update các gói phần mềm và grafana

```sh
apt-get update
apt-get -y install grafana
```

- Bước 5: Khởi động dịch vụ

```sh
service grafana-server start
```

- Bước 6: Cho grafana khởi động cùng hệ thống

```sh
update-rc.d grafana-server defaults 95 10
```

- Bước 7: Truy cập vào giao diện web như sau

```sh
http://<Grafana-Server-IP>:3000/
```

Màn hình khi login

<img src="http://i.imgur.com/Y0uDbzt.png">

<a name="23"></a>
### 2.3 Cài đặt trên Centos

- Bước 1: Tải các gói cài đặt và giải nén

```sh
wget https://s3-us-west-2.amazonaws.com/grafana-releases/release/grafana-4.2.0-1.x86_64.rpm
yum install initscripts fontconfig
rpm -Uvh grafana-4.2.0-1.x86_64.rpm
```

- Bước 2: Thêm cấu hình vào file `/etc/yum.repos.d/grafana.repo`

```sh
vi /etc/yum.repos.d/grafana.repo


[grafana]
name=grafana
baseurl=https://packagecloud.io/grafana/stable/el/6/$basearch
repo_gpgcheck=1
enabled=1
gpgcheck=1
gpgkey=https://packagecloud.io/gpg.key https://grafanarel.s3.amazonaws.com/RPM-GPG-KEY-grafana
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
```

- Bước 3: Cài đặt Grafana

```sh
yum install grafana -y
```

- Bước 4: Khởi động Grafana và cho dịch vụ khởi động cùng hệ thống

```sh
service grafana-server start

/sbin/chkconfig --add grafana-server
```

<a name="3"></a>
### 3. Hướng dẫn sử dụng dashboard grafana

<a name="31"></a>
### 3.1 Thêm Plugin Zabbix vào Grafana

Để thêm Plugin Zabbix vào Grafana thực hiện như sau

- Bước 1: Trên giao diện Web của Grafana chọn `Install apps & plugins`

<img src="http://i.imgur.com/vAuNIYh.png">

- Bước 2: Trong danh sách các ứng dụng, tìm Zabbix Plugin

<img src="http://i.imgur.com/yml0UAm.png">

- Bước 3: Cài đặt Zabbix Plugin

<img src="http://i.imgur.com/y0NuDhp.png">

<ul>
<li>Ta có thể chọn phiên bản Zabbix plugin trước khi cài đặt</li>
<li>Thực hiện câu lệnh sau trên Grafana server để cài đặt Plugin</li>
</ul>

```sh
grafana-cli plugins install alexanderzobnin-zabbix-app
```

Khởi động lại dịch vụ

```sh
service grafana-server restart
```

- Bước 4: Cài đặt Zabbix Plugin trên Web Grafana

Để cài đặt chọn toogle bar -> Plugins -> Zabbix

<img src="http://i.imgur.com/XupHSAF.png">

- Bước 5: Enable dịch vụ

<img src="http://i.imgur.com/i29p0Iy.png">

Sau đó quay trở lại trang Home của Grafana kiểm tra

<img src="http://i.imgur.com/UEnVFUo.png">


<a name="32"></a>
### 3.2 Thêm Data Sources

- Bước 1: Trên toogle bar chọn `Data Sources`

<img src="http://i.imgur.com/JDZHkkn.png">

- Bước 2:  Add Data Sources

<img src="http://i.imgur.com/Ln4Y3Rj.png">

- Bước 3: Khai báo các thông số sau đây

<img src="http://i.imgur.com/LygYmxN.png">

<ul>
<li>Khai báo tên của Data Sources</li>
<li>Chọn type là Zabbix</li>
<li>Khai báo địa chỉ IP Server cần lấy dữ liệu</li>
<li>Nhập thông tin đăng nhập Zabbix</li>
</ul>

Chọn Save and Test


<a name="4"></a>

## 4. Tạo 1 dashboard cơ bản

Tạo 1 Dashboard để giám sát host với Plugin Zabbix. Để có thể tạo 1 dashboard giám sát host với Plugin Zabbix cần phải cài đặt thành công Zabbix Server và giám sát được host. Tham khảo cài đặt Zabbix tại [đây](https://github.com/trimq/mdt-technical/blob/master/TRIMQ/Zabbix/docs/Caidat-Zabbix.md)

- Bước 1: Tạo Dashboard

<img src="http://i.imgur.com/W2SNj05.png">

- Bước 2: Lưu lại Dashboard vừa tạo và đặt tên

<img src="http://i.imgur.com/F9gv942.png">


<a name="41">
4.1 Singlestat

Singlestat sử dụng để hiển thị các thông số đơn như số User login, System Uptime, Server status...

- Tạo biểu đồ:

<img src="http://i.imgur.com/aVd1HnE.png">


- Chỉnh sửa biểu đồ:

<img src="http://i.imgur.com/FFJ6nXE.png">

- Đặt tên cho biểu đồ

<img src="">

- Thêm metric cho biểu đồ

### Lưu ý: Các metric này chính là các item của Zabbix. Trên Zabbix bắt buộc phải có những item này thì Grafana mới nhận được dữ liệu để đưa ra biểu đồ















