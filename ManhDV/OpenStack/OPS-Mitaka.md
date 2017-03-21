#Hu?ng d?n cài d?t OpenStack trên Mitaka v?i mô hình Openvswitch

# 1. Chu?n b?

 -	Distro : RHEL 7 dã register

 -	Mô hình
 
![ops](/ManhDV/OpenStack/images/ops-ovs-system.png)

 -	IP Planing

![ops](/ManhDV/OpenStack/images/ipplan-01.png)

# 2. Setup môi tru?ng cài d?t (Trên c? CTL và COM)

 -	C?u hình file host
 
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

 - Ki?m tra ping ra Internet
 
`ping google.com`

 - Config cho các module network
 
```sh
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
echo "net.ipv4.conf.all.rp_filter=0" >> /etc/sysctl.conf
echo "net.ipv4.conf.default.rp_filter=0" >> /etc/sysctl.conf
sysctl -p
```

 - Ðang ký tài kho?n RHEL (Ngu?i dùng dã ph?i dang ký b?ng mail trên website c?a RHEL)
 
`subscription-manager register --username="user" --password="userpassword" --auto-attach`

`subscription-manager repos --enable=rhel-7-server-optional-rpms --enable=rhel-7-server-extras-rpms --enable=rhel-7-server-rh-common-rpms`

 - T?t firewall và selinux
```sh
systemctl disable firewalld
systemctl stop firewalld
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
```

 - Kh?i d?ng l?i máy
 
`init 6`


 -  Add repo và update h? th?ng

`yum install -y https://repos.fedorapeople.org/repos/openstack/openstack-mitaka/rdo-release-mitaka-6.noarch.rpm`

`yum upgrade -y`

 - Cài d?t byobu
 
`yum install -y http://dl.fedoraproject.org/pub/epel/7/x86_64/b/byobu-5.73-4.el7.noarch.rpm`

 - Cài d?t openstack-client d? s? d?ng các câu l?nh openstack
 
`yum install python-openstackclient -y`

 - Cài d?t gói openstack-selinux d? qu?n lý policy cho các service openstack.
 
`yum install openstack-selinux -y`

## 2.1 Cài d?t và c?u hình d?ch v? d?ng b? th?i gian NTP

 - Cài d?t NTP
`yum install chrony -y `

 - Ch?nh s?a file c?u hình /etc/chrony/chrony.conf
 
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
 - Restart d?ch v? ntp
 
```sh
systemctl enable chronyd.service
systemctl start chronyd.service
```
 - Ch?y l?nh ki?m tra trên 2 node CTL và COM
 
`chronyc sources`

# 3. Cài d?t trên node Controller
##3.1 Cài d?t và c?u hình database MySQL

 - Cài d?t database MySQL

`yum install -y mariadb mariadb-server python2-PyMySQL `

 - T?o file c?u hình cho d?ch v? Openstack 
 
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

 - Start d?ch v? và cho phép kh?i d?ng d?ch v? khi kh?i d?ng máy.
 
```sh
systemctl enable mariadb.service
systemctl start mariadb.service
```

 - Th?c hi?n security cho mysql, th?c hi?n theo các bu?c sau : 
 
`mysql_secure_installation`

```sh
Enter current password for root (enter for none): [your password]
Change the root password? [Y/n]: n
Remove anonymous users? [Y/n]: y
Disallow root login remotely? [Y/n]: y
Remove test database and access to it? [Y/n]: y
Reload privilege tables now? [Y/n]: y
```

## 3.2 Cài d?t và c?u hình RabbitMQ

 - Cài d?t rabbitmq
 
`yum install rabbitmq-server -y `

 - Start d?ch v? và cho phép kh?i d?ng d?ch v? khi kh?i d?ng máy
 
```sh
systemctl enable rabbitmq-server.service
systemctl start rabbitmq-server.service
```

 - Thêm user **openstack**
 
`rabbitmqctl add_user openstack Welcome123`

 - Phân quy?n cho user **openstack** du?c phép config, write, read trên rabbitmq

`rabbitmqctl set_permissions openstack ".*" ".*" ".*"`

## 3.3. Cài d?t và c?u hình Memcache

 - Cài d?t memcache
 
`yum install -y memcached python-memcached`

 - Sao luu c?u hình memcache
 
`cp /etc/sysconfig/memcached /etc/sysconfig/memcached.bka`

 - Chính s?a c?u hình memcache
 
`vi /etc/sysconfig/memcached`

```sh
PORT="11211"
USER="memcached"
MAXCONN="1024"
CACHESIZE="64"
OPTIONS="-l 0.0.0.0,::1"
```
 - Start d?ch v? và cho phép kh?i d?ng d?ch v? khi kh?i d?ng máy
 
```sh
systemctl enable memcached.service
systemctl start memcached.service
```



 

  
  