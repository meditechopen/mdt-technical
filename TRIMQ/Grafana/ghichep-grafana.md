# Công cụ Grafana

# Mục lục

- [1. Giới thiệu về Grafana](#1)
	- [1.1 Một số thuật ngữ dùng trong Grafana](#11)
- [2. Cài đặt grafana](#2)
	- [2.1 Cài đặt trên Ubuntu](#21)
	- [2.2 Cài đặt trên Centos](#22)
- [3. Hướng dẫn sử dụng dashboard grafana](#3)
- [4. Tạo 1 dashboard cơ bản](#4)
- [5. Tài liệu tham khảo](#5)

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

Cài đặt Grafana trên các distro linux khác nhau. Phiên bản ổn định mới nhất đang là Grafana 4.2.0 (x86-64 deb)


<a name="21"></a>
### 2.1 Cài đặt trên Ubuntu

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

<a name="22"></a>
### 2.2 Cài đặt trên Centos

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




