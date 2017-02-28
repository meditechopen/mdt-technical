# Cấu hình ISCSI-LVM Backup

# Mục lục:
- [1. Mô hình triển khai](#1)
- [2. Cấu hình ISCSI](#2)
	- [2.1. Trên máy ISCSI-Target](#21)
	- [2.2 Trên máy App1](#22)
	- [2.3 Trên máy App2](#23)
- [3. Backup LVM](#3)
- [4. Tài liệu tham khảo](#4)

====================================

<a name="1"></a>
## 1. Mô hình triển khai
3 Máy Server chạy CentOS 6.5

- Máy ISCSI-Target (3 ổ cứng): Nhiệm vụ cung cấp storage cho các máy còn lại
<ul>
<li>sda: Cài OS</li>
<li>sdb: Cấp storage</li>
<li>sdc: Cấp storage</li>
</ul>

- Máy App1 (1 ổ cứng cài OS)

- Máy App2 (1 ổ cứng cài OS)

- Mô hình:
<img src="">

<a name="2"></a>
## 2. Cấu hình ISCSI

<a name="21"></a>
### 2.1 Trên máy ISCSI-Target:

- Cài đặt gói ISCSI-Target
```sh
yum install scsi-target-utils -y
```

- Chỉnh sửa file cấu hình `/etc/tgt/targets.conf`. Thêm những dòng sau vào cuối file

```sh
<target iqn.2017-02.hanu.vn:target01>
backing-store /dev/sdb
initiator-address 192.168.100.31
incominguser root Welcome123
</target>

<target iqn.2017-02.hanu.vn:target02>
backing-store /dev/sdc
initiator-address 192.168.100.31
incominguser root Welcome123
</target>
```

- Khởi động dịch vụ
```sh
/etc/rc.d/init.d/tgtd start
```

- Xác thực lại trạng thái cấp storage
```sh
tgtadm --mode target --op show
```

<a name="22"></a>
### 2.2 Trên máy App1
- Cài đặt dịch vụ iscsi-initiator
```sh
yum install iscsi-initiator-utils -y
```

- Sửa lại file `/etc/iscsi/initiatorname.iscsi`
```sh
InitiatorName=iqn.2017-02.hanu.vn:initiator
```

- Sửa lại file `/etc/iscsi/iscsid.conf`
```sh
uncomment dòng 56
Dòng 60, 61 uncomment và nhập thông tin user máy iscsi-initiator
```

- Khởi động lại dịch vụ
```sh
service iscsid force-start
```

- Kiểm tra kết nối iscsi-initiator đến target
```sh
telnet 192.168.100.30 3260
```

- Nếu iscsi-initiator không kết nối được đến target, kiểm tra iptables trên máy ISCSI-Target


- Tìm ISCSI-Target
```sh
iscsiadm -m discovery -t sendtargets -p 192.168.100.30
```

- Đăng nhập vào máy ISCSI-Target
```sh
iscsiadm -m node --login
```

- Kiểm tra ổ cứng trên App1, nếu xuất hiện 2 ổ cứng sdb và sdc là thành công
```sh
lsblk
```

### Đứng trên App1 cài đặt LVM
- Phân vùng 2 ổ đĩa sdb và sdc sử dụng `fdisk`

- Tạo ra các Physical Volume từ 2 phân vùng trên
```sh
pvcreate /dev/sdb1
pvcreate /dev/sdc1
```

- Tạo Volume Group có tên là vg1
```sh
vgcreate vg1 /dev/sdb1 /dev/sdc1
```

- Tạo Logical Volume lv1 trên Volume Group
```sh
lvcreate -L 1G -n lv1 vg1
```

- Format logical volume đã tạo
```sh
mkfs.ext4
```

- Mount vào sử dụng
```sh
mount /dev/vg1/lv1 /mnt
```

- Đẩy dữ liệu vào thư mục `/mnt` để kiểm tra

- <b>Sao lưu file backup của lvm trên máy App1 ra máy khác</b>

## Trường hợp đặt ra là máy App1 không hoạt động và sử dụng App2 để connect tới ISCSI-Target để backup lại LVM trên đó

<a name="23"></a>
### 2.3 Trên máy App2
- Cài đặt dịch vụ iscsi-initiator
```sh
yum install iscsi-initiator-utils -y
```

- Sửa lại file `/etc/iscsi/initiatorname.iscsi`
```sh
InitiatorName=iqn.2017-02.hanu.vn:initiator
```

- Sửa lại file `/etc/iscsi/iscsid.conf`
```sh
uncomment dòng 56
Dòng 60, 61 uncomment và nhập thông tin user máy iscsi-initiator
```

- Khởi động lại dịch vụ
```sh
service iscsid force-start
```

### Sửa lại file cấu hình trên ISCSI-Target và khởi động lại dịch vụ

```sh
<target iqn.2017-02.hanu.vn:target01>
backing-store /dev/sdb
initiator-address 192.168.100.32
incominguser root Welcome123
</target>

<target iqn.2017-02.hanu.vn:target02>
backing-store /dev/sdc
initiator-address 192.168.100.32
incominguser root Welcome123
```


- Kiểm tra kết nối iscsi-initiator đến target
```sh
telnet 192.168.100.30 3260
```

- Kiểm tra ổ đĩa trên máy
```sh
lsblk
```

- Kiểm tra các Physical Volume
```sh
pvscan
```

- Kiểm tra các Volume group
```sh
vgscan
```

- Backup lại volume group
```sh
vgcfgbackup -f /etc/lvm/backup/lvm1/backup/vg1
```

- Restore lại volume group
```sh
vgcfgrestore vg1 -f /etc/lvm/backup/lvm1/backup/vg1
```

- Active lại logical volume đã tồn tại trên volume group
```sh
lvchange -a y /dev/vg1/lv1
```

- Mount lại vào và sử dụng
```sh
mount /dev/vg1/lv1 /mnt
```

- Kiểm tra lại những dữ liệu đã tồn tại trên LVM

<a name="4"></a>
## 4. Tài liệu tham khảo

- (1)https://www.centos.org/docs/5/html/Cluster_Logical_Volume_Manager/active_mount_ex3.html

- (2)http://www.techrepublic.com/blog/linux-and-open-source/use-vgcfgbackup-and-vgcfgrestore-to-back-up-metadata-on-lvm/

- (3)http://blogit.edu.vn/cai-dat-scsi-target-va-device-mapper-multipath-tren-centos-6/
