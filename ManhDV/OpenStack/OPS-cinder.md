# Cài đặt và cấu hình dịch vụ Cinder

# 1. Cài đặt và cấu hình Cinder trên node Controller

 - Tạo database cho Cinder 
 
```sh
 mysql -u root -pWelcome123
CREATE DATABASE cinder;
GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'localhost' \
  IDENTIFIED BY 'Welcome123';
GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'%' \
  IDENTIFIED BY 'Welcome123';
exit
```

 - Tạo và phân quyền cho user cinder 
 
`source admin-rc`
 
`openstack user create cinder --domain default --password Welcome123`

`openstack role add --project service --user cinder admin`

 - Tạo service entity 
 
```sh
openstack service create --name cinder --description "OpenStack Block Storage" volume
  
openstack service create --name cinderv2 --description "OpenStack Block Storage" volumev2
```

 - Tạo service API endpoint
 
```sh
openstack endpoint create --region RegionOne volume public http://controller:8776/v1/%\(tenant_id\)s
  
openstack endpoint create --region RegionOne volume internal http://controller:8776/v1/%\(tenant_id\)s

openstack endpoint create --region RegionOne volume admin http://controller:8776/v1/%\(tenant_id\)s

openstack endpoint create --region RegionOne volumev2 public http://controller:8776/v2/%\(tenant_id\)s

openstack endpoint create --region RegionOne volumev2 internal http://controller:8776/v2/%\(tenant_id\)s

openstack endpoint create --region RegionOne volumev2 admin http://controller:8776/v2/%\(tenant_id\)s
```

 - Cài đặt Cinder
 
`yum install -y openstack-cinder`

 - Sao lưu file cấu hình

`cp /etc/cinder/cinder.conf /etc/cinder/cinder.conf.orig`

```sh
[DEFAULT]
rpc_backend = rabbit
auth_strategy = keystone
my_ip = 172.16.69.11

[database]
connection = mysql+pymysql://cinder:Welcome123@controller/cinder

[oslo_messaging_rabbit]
rabbit_host = controller
rabbit_userid = openstack
rabbit_password = Welcome123

[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = cinder
password = Welcome123

[oslo_concurrency]
lock_path = /var/lib/cinder/tmp
```

 
 - Đồng bọ database Cinder
 
`su -s /bin/sh -c "cinder-manage db sync" cinder`

 - Sửa file /etc/nova/nova.conf
 
```sh
[cinder]
os_region_name = RegionOne
```

 - Restart dịch vụ nova
 
```sh
systemctl restart openstack-nova-api.service
```

 - Restart dịch vụ cinder và cho phép khởi động cùng hệ thống :

``sh
systemctl enable openstack-cinder-api.service openstack-cinder-scheduler.service
systemctl start openstack-cinder-api.service openstack-cinder-scheduler.service
```

# Cài đặt và cấu hình cinder trên Cinder node

 - Cài đặt LVM
 
`yum install -y lvm2`

 - Khởi động dịch vụ LVM và cho phép khởi động cùng hệ thống.
 
```sh
systemctl enable lvm2-lvmetad.service
systemctl start lvm2-lvmetad.service
```

 - Tạo LVM physical volume /dev/sdb
 
`pvcreate /dev/sdb`

 - Tạo LVM volume group `cinder-volumes`
 
`vgcreate cinder-volumes /dev/sdb`

 - Sửa file /etc/lvm/lvm.conf, để LVM chỉ scan và dùng ổ sba cho OS và sdb cho block storage
 
```sh
devices {
...
filter = [ "a/sda/", "a/sdb/", "r/.*/"]
```

 - Cài đặt package cho cinder
 
`yum install openstack-cinder targetcli python-keystone`

 - Sao lưu file cấu hình cinder /etc/cinder/cinder.conf 
 
```sh
[DEFAULT]
rpc_backend = rabbit
auth_strategy = keystone
my_ip = 192.168.11.12
enabled_backends = lvm
glance_api_servers = http://172.16.69.11:9292

[oslo_messaging_rabbit]
rabbit_host = controller
rabbit_userid = openstack
rabbit_password = Welcome123

[database]
connection = mysql+pymysql://cinder:Welcome123@controller/cinder

[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = cinder
password = Welcome123

[lvm]
volume_driver = cinder.volume.drivers.lvm.LVMVolumeDriver
volume_group = cinder-volumes
iscsi_protocol = iscsi
iscsi_helper = lioadm

[oslo_concurrency]
lock_path = /var/lib/cinder/tmp

 - Start dịch vụ cinder-volume và cho phép khởi động cùng hệ thống 
 
```sh
systemctl enable openstack-cinder-volume.service target.service
systemctl start openstack-cinder-volume.service target.service
```

 - Quay lại node CTL, kiểm tra cinder-list
 
```sh
source admin-rc
cinder service-list
```