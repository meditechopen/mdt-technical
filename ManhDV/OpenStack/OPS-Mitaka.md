#Hướng dẫn cài đặt OpenStack trên Mitaka với mô hình Openvswitch

# 1. Chuẩn bị

 -	Distro : RHEL 7 đã register

 -	Mô hình
 
![ops](/ManhDV/OpenStack/images/ops-ovs-system.png)

 -	IP Planing

![ops](/ManhDV/OpenStack/images/ipplan-01.png)

# 2. Setup môi trường cài đặt (Trên cả CTL và COM)

 -	Cấu hình file host
 
`vi /etc/hosts`
 
```sh
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
172.16.69.10    controller
172.16.69.20    compute
```

`vi /etc/resolv.conf`

```sh
nameserver 8.8.8.8
```

 - Kiểm tra ping ra Internet
 
`ping google.com`

 - Config cho các module network
 
```sh
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
echo "net.ipv4.conf.all.rp_filter=0" >> /etc/sysctl.conf
echo "net.ipv4.conf.default.rp_filter=0" >> /etc/sysctl.conf
sysctl -p
```

 - Đăng ký tài khoản RHEL (Người dùng đã phải đăng ký bằng mail trên website của RHEL)
 
`subscription-manager register --username="user" --password="userpassword" --auto-attach`

`subscription-manager repos --enable=rhel-7-server-optional-rpms --enable=rhel-7-server-extras-rpms --enable=rhel-7-server-rh-common-rpms`

 - Tắt firewall và selinux
```sh
systemctl disable firewalld
systemctl stop firewalld
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
```

 - Khởi động lại máy
 
`init 6`


 -  Add repo và update hệ thống

`yum install -y https://repos.fedorapeople.org/repos/openstack/openstack-mitaka/rdo-release-mitaka-6.noarch.rpm`

`yum upgrade -y`

 - Cài đặt byobu
 
`yum install -y http://dl.fedoraproject.org/pub/epel/7/x86_64/b/byobu-5.73-4.el7.noarch.rpm`

 - Cài đặt openstack-client để sử dụng các câu lệnh openstack
 
`yum install python-openstackclient -y`

 - Cài đặt gói openstack-selinux để quản lý policy cho các service openstack.
 
`yum install openstack-selinux -y`

## 2.1 Cài đặt và cấu hình dịch vụ đồng bộ thời gian NTP

 - Cài đặt NTP
`yum install chrony -y `

 - Chỉnh sửa file cấu hình /etc/chrony/chrony.conf
 
```sh
sed -i "s/server 0.debian.pool.ntp.org offline minpoll 8/ \
server 172.16.69.10 iburst/g" /etc/chrony/chrony.conf

sed -i 's/server 1.debian.pool.ntp.org offline minpoll 8/ \
# server 1.debian.pool.ntp.org offline minpoll 8/g' /etc/chrony/chrony.conf

sed -i 's/server 2.debian.pool.ntp.org offline minpoll 8/ \
# server 2.debian.pool.ntp.org offline minpoll 8/g' /etc/chrony/chrony.conf

sed -i 's/server 3.debian.pool.ntp.org offline minpoll 8/ \
# server 3.debian.pool.ntp.org offline minpoll 8/g' /etc/chrony/chrony.conf

sed -i 's// \
 /g'/etc/chrony/chrony.conf 
```
 - Restart dịch vụ ntp
 
```sh
systemctl enable chronyd.service
systemctl start chronyd.service
```
 - Chạy lệnh kiểm tra trên 2 node CTL và COM
 
`chronyc sources`

# 3. Cài đặt trên node Controller
##3.1 Cài đặt và cấu hình database MySQL

 - Cài đặt database MySQL

`yum install -y mariadb mariadb-server python2-PyMySQL `

 - Tạo file cấu hình cho dịch vụ Openstack 
 
`vi /etc/my.cnf.d/openstack.cnf`

```sh
[mysqld]

bind-address = 172.16.69.10
default-storage-engine = innodb
innodb_file_per_table
max_connections = 4096
collation-server = utf8_general_ci
character-set-server = utf8
```

 - Start dịch vụ và cho phép khởi động dịch vụ khi khởi động máy.
 
```sh
systemctl enable mariadb.service
systemctl start mariadb.service
```

 - Thực hiện security cho mysql, thực hiện theo các bước sau : 
 
`mysql_secure_installation`

```sh
Enter current password for root (enter for none): [your password]
Change the root password? [Y/n]: n
Remove anonymous users? [Y/n]: y
Disallow root login remotely? [Y/n]: y
Remove test database and access to it? [Y/n]: y
Reload privilege tables now? [Y/n]: y
```

## 3.2 Cài đặt và cấu hình RabbitMQ

 - Cài đặt rabbitmq
 
`yum install rabbitmq-server -y `

 - Start dịch vụ và cho phép khởi động dịch vụ khi khởi động máy
 
```sh
systemctl enable rabbitmq-server.service
systemctl start rabbitmq-server.service
```

 - Thêm user **openstack**
 
`rabbitmqctl add_user openstack Welcome123`

 - Phân quyền cho user **openstack** được phép config, write, read trên rabbitmq

`rabbitmqctl set_permissions openstack ".*" ".*" ".*"`

## 3.3. Cài đặt và cấu hình Memcache

 - Cài đặt memcache
 
`yum install -y memcached python-memcached`

 - Sao lưu cấu hình memcache
 
`cp /etc/sysconfig/memcached /etc/sysconfig/memcached.bka`

 - Chính sửa cấu hình memcache
 
`vi /etc/sysconfig/memcached`

```sh
PORT="11211"
USER="memcached"
MAXCONN="1024"
CACHESIZE="64"
OPTIONS="-l 0.0.0.0,::1"
```
 - Start dịch vụ và cho phép khởi động dịch vụ khi khởi động máy
 
```sh
systemctl enable memcached.service
systemctl start memcached.service
```



 

  
  