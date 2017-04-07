# Cấu hình Gluster FS

# Mục lục:

- [1. Giới thiệu Gluster FS](#1)
	- [1.1 Giới thiệu Gluster FS](#11)
	- [1.2 Thuật ngữ trong Gluster FS](#12)
	- [1.3 Một số loại volume cơ bản](#13)
- [2. IP Planing & Topology lab](#2)
	- [2.1 IP Planing](#21)
	- [2.2 Topology lab](#22)
- [3. Dựng lab Gluster](#3)
	- [3.1 Các bước cấu hình trên server](#31)
	- [3.2 Các bước cấu hình trên client](#32)
	- [3.3 Kiểm tra khả năng hoạt động của Gluster FS](#33)
- [4. Khác](#4)
- [5. Tài liệu tham khảo](#5)

-------------------------------------------------------------------------------

<a name="1"></a>
## 1. Giới thiệu Gluster FS

<a name="11"></a>
### 1.1 Giới thiệu Gluster FS
- GlusterFS là một open source, là tập hợp file hệ thống có thể được nhân rộng tới vài peta-byte và có thể xử lý hàng ngàn Client.

- GlusterFS có thể linh hoạt kết hợp với các thiết bị lưu trữ vật lý, ảo, và tài nguyên điện toán đám mây để cung cấp 1 hệ thống lưu trữ có tính sẵn sàng cao và khả năng performant cao .

- Chương trình có thể lưu trữ dữ liệu trên các mô hình, thiết bị khác nhau, nó kết nối với tất cả các nút cài đặt GlusterFS qua giao thức TCP hoặc RDMA tạo ra một nguồn tài nguyên lưu trữ duy nhất kết hợp tất cả các không gian lưu trữ có sẵn thành một khối lượng lưu trữ duy nhất (distributed mode) hoặc sử dụng tối đa không gian ổ cứng có sẵn trên tất cả các ghi chú để nhân bản dữ liệu của bạn (replicated mode).


<a name="12"></a>
### 1.2 Thuật ngữ trong Gluster FS

Để có thể hiểu rõ về GlusterFS và ứng dụng được sản phẩm này, trước hết ta cần phải biết rõ những khái niệm có trong GlusterFS. Sau đây là những khái niệm quan trọng khi sử dụng Glusterfs

- <b>Trusted Storage Pool</b>: Trong một hệ thống GlusterFS, những server dùng để lưu trữ được gọi là những node, và những node này kết hợp lại với nhau thành một không gian lưu trữ lớn được gọi là Pool. Dưới đây là mô hình kết nối giữa 2 node thành một Trusted Storage Pool.

<img src="https://camo.githubusercontent.com/88933752e8ec0e3220e1a6b2b5e4051fcb495895/687474703a2f2f692e696d6775722e636f6d2f57757842536e5a2e706e67">

- <b>Brick</b>: 
<ul>
<li>Từ những phần vùng lưu trữ mới (những phân vùng chưa dùng đến) trên mỗi node, chúng ta có thể tạo ra những brick.</li>
<li>Brick được định nghĩa bởi 1 server (name or IP) và 1 đường dẫn. Vd: 10.10.10.20:/mnt/brick (đã mount 1 partition (/dev/sdb1) vào /mnt)</li>
<li>Mỗi brick có dung lượng bị giới hạn bởi filesystem....</li>
<li>Trong mô hình lý tưởng, mỗi brick thuộc cluster có dung lượng bằng nhau. Để có thể hiểu rõ hơn về Bricks, chúng ta có thể tham khảo hình dưới đây:</li>

<img src="https://camo.githubusercontent.com/8a8573b20d405d08b0760b6f2950a542f49b9672/687474703a2f2f692e696d6775722e636f6d2f7645766d304a372e706e67"> 
</ul>

- <b>Volume</b>:
<ul>
<li>Từ những brick trên các node thuộc cùng một Pool, kết hợp những brick đó lại thành một không gian lưu trữ lớn và thống nhất để client có thể mount đến và sử dụng.</li>
<li>Một volume là tập hợp logic của các brick. Tên volume được chỉ định bởi administrator</li>
<li>Volume được mount bởi client: mount -t glusterfs server1:/ /my/mnt/point </li>
<li>Một volume có thể chứa các brick từ các node khác nhau. Sau đây là mô hình tập hợp những Brick thành Volume:</li>

</ul>

<img src="https://camo.githubusercontent.com/58b199b4372fe97ebe7872bb3f2536a9e2503e9f/687474703a2f2f692e696d6775722e636f6d2f53676f6c5654712e706e67">

Tại hình trên, chúng ta có thể thấy mỗi Node1, Node2, Node3 đã tạo 2 brick là /export/brick1 và /export/brick2, và từ 3 brick /export/brick1trên 3 Node tập hợp lại tạo thành volume music. Tương tự 3 brick /export/brick2 trên 3 Node tập hợp lại tạo thành volume Videos.

<a name="13"></a>
### 1.3 Một số loại volume cơ bản

Khi sử dụng GlusterFS có thể tạo nhiều loại volume và mỗi loại có được những tính năng khác nhau. Dưới đây là 5 loại volume cơ bản

#### Distributed volume:

Distributed Volume có những đặc điểm cơ bản sau:

Dữ liệu được lưu trữ phân tán trên từng bricks, file1 nằm trong brick 1, file 2 nằm trong brick 2,...

Vì metadata được lưu trữ trực tiếp trên từng bricks nên không cần thiết phải có một metadata server ở bên ngoài, giúp cho các tổ chức tiết kiệm được tài nguyên.

Ưu điểm: mở rộng được dung lượng store ( dung lượng store bằng tổng dung lượng các brick)

Nhược điểm: nếu 1 trong các brick bị lỗi, dữ liệu trên brick đó sẽ mất


<img src="https://camo.githubusercontent.com/c5aa90c4727697e1c57c7c8d34f5613688baca1b/687474703a2f2f692e696d6775722e636f6d2f5a41366438664f2e706e67">


#### Replicated volume:

Dữ liệu sẽ được nhân bản đến những brick còn lại, trên tất cả các node và đồng bộ tất cả các nhân bản mới cập nhật.

Đảm bảo tính nhất quán.

Không giới hạn số lượng replicas.

Ưu điểm: phù hợp với hệ thống yêu cầu tính sẵn sàng cao và dự phòng

Nhược điểm: tốn tài nguyên hệ thống

<img src="https://camo.githubusercontent.com/ba8d6de501a20aaa16771da19801d13a5a1448b9/687474703a2f2f692e696d6775722e636f6d2f48396d73424e482e706e67">

#### Stripe volume:

Dữ liệu chia thành những phần khác nhau và lưu trữ ở những brick khác nhau, ( 1 file được chia nhỏ ra trên các brick )

Ưu điểm : phù hợp với những môi trường yêu cầu hiệu năng, đặc biệt truy cập những file lớn.

Nhược điểm: 1 brick bị lỗi volume không thể hoạt động được.

<img src="https://camo.githubusercontent.com/f8cf164c946b6259516e2fa94e22599c1b52f566/687474703a2f2f692e696d6775722e636f6d2f6e50496e59656e2e706e67">

#### Distributed replicated:

Kết hợp từ distributed và replicated

<img src="https://camo.githubusercontent.com/9e56b066833920595fcd9fa528cc80e4add1e782/687474703a2f2f692e696d6775722e636f6d2f62454f746753372e706e67">

Với mô hình trên, hệ thống sẽ yêu cầu cần tối thiểu 3 node, vừa có thể mở rộng được dung lượng lưu trữ, vừa tăng tính dự phòng cho hệ thống. Tuy nhiên, nếu đồng thời bị lỗi 2 node server1 và server2 hoặc 2 node server3 và server4 thì hệ thống sẽ không hoạt động được.

#### Distributed stripe volume:

Kết hợp từ Distributed và stripe. Do đó nó có hầu hết những thuộc tính hai loại trên và khi 1 node và 1 brick delete đồng nghĩa volume cũng không thể hoạt động được nữa.

<img src="https://camo.githubusercontent.com/7fab4d577e6547eb90d95ca0e0093cdbe2a50b82/687474703a2f2f692e696d6775722e636f6d2f765236463761322e706e67">

#### Replicated stripe volume

Kết hợp từ replicated và stripe

<img src="https://camo.githubusercontent.com/eeb943f4fdd10cb979f422035cc3f263dac64488/687474703a2f2f692e696d6775722e636f6d2f6e52696a754a792e706e67">











<a name="2"></a>
## 2. IP Planing & Topology lab

Sau đây là IP Planing và Topology lab 

<a name="21"></a>
### 2.1 IP Planing

| Host | OS | IP | Disk 1 | Disk 2 | Hostname |
|------|----|----|--------|--------|----------|
| Server 01 | Ubuntu 14 | 192.168.100.50 | sda 20G cài OS | sdb 10G trống | gluster01 |
| Server 02 | Ubuntu 14 | 192.168.100.51 | sda 20G cài OS | sdb 10G trống | gluster02 |
| Client | Ubuntu 14 | 192.168.100.42 | sda 20G cài OS | Không có | grafana |

<a name="22"></a>
### 2.2 Topology lab

<img src="http://i.imgur.com/5cclFtW.png">


<a name="3"></a>
### 3. Dựng lab Gluster

### Thực hiện các bước sau trên Server01

- Bước 1: Khai báo trong file `/etc/hosts` như sau

```sh
127.0.0.1       localhost
127.0.1.1       gluster01
192.168.100.51  gluster02
192.168.100.42  grafana
```

Bước khai báo trên giúp các node trong hệ thống có thể ping đến nhau thông qua hostname

- Bước 2: Phân vùng cho ổ cứng

```sh
fdisk /dev/sdb
```

- Bước 3: Format phân vùng định dạng xfs. Trong 1 số trường hợp không có sẵn gói định dạng thì phải tải về

```sh
apt-get install xfsprogs -y


mkfs.xfs /dev/sdb1
```

- Bước 4: Mount partition vào thư mục /mnt và tạo thư mục /mnt/brick1

```sh
mount /dev/sdb1 /mnt && mkdir -p /mnt/brick1
```

- Bước 5: Khai báo vào file cấu hình /etc/fstab để khi restart server, hệ thống sẽ tự động mount vào thư mục.

```sh
echo "/dev/sdb1 /mnt xfs defaults 0 0" >> /etc/fstab
```

- Bước 6: Tải gói gluster server

```sh
apt-get install glusterfs-server -y
```

### Thực hiện các bước sau trên Server02

- Bước 2: Phân vùng cho ổ cứng

```sh
fdisk /dev/sdb
```

- Bước 3: Format phân vùng định dạng xfs. Trong 1 số trường hợp không có sẵn gói định dạng thì phải tải về

```sh
apt-get install xfsprogs -y


mkfs.xfs /dev/sdb1
```

- Bước 4: Mount partition vào thư mục /mnt và tạo thư mục /mnt/brick1

```sh
mount /dev/sdb1 /mnt && mkdir -p /mnt/brick1
```

- Bước 5: Khai báo vào file cấu hình /etc/fstab để khi restart server, hệ thống sẽ tự động mount vào thư mục.

```sh
echo "/dev/sdb1 /mnt xfs defaults 0 0" >> /etc/fstab
```

- Bước 6: Tải gói gluster server

```sh
apt-get install glusterfs-server -y
```

- Bước 7: Tạo 1 pool storage với Server gluster01

```sh
gluster peer probe 192.168.100.50
```

- Bước 8: Kiểm tra trạng thái của gluster pool

```sh
gluster peer status


OUTPUT:
Number of Peers: 1

Hostname: 192.168.100.51
Port: 24007
Uuid: f2841a02-d536-4044-93c7-a5c7a029178e
State: Peer in Cluster (Connected)
```

Như vậy tôi đã tạo 1 pool với 2 brick từ 2 node storage 

- Bước 9: Sau đây tôi sẽ tạo 1 Volume Replicated 

```sh
gluster volume create testvol2 rep 2 transport tcp 192.168.100.50:/mnt/brick1 192.168.100.51:/mnt/brick1
```

#### Giải thích cú pháp lệnh:

| Cú pháp | Ý nghĩa |
|---------|---------|
| gluster volume create | Câu lệnh để tạo volume |
| testvol2 | Tên volume |
| rep | Loại volume là replicated |
| 2 | Số brick |
| transport tcp | Giao thức để liên lạc là tcp |
| 192.168.100.50:/mnt/brick1 | Đường dẫn của brick |
| 192.168.100.51:/mnt/brick1 | Đường dẫn của brick |

- Bước 10: Sau khi tạo volume, tôi sẽ khởi động volume đó lên. Bước này có thể thực hiện trên cả 2 Server gluster

```sh
gluster volume start testvol2
```

<a name="32"></a>
### 3.2 Các bước cấu hình trên client

- Bước 1: Tải gói gluster client

```sh
apt-get install glusterfs-client -y
```

- Bước 2: Mount volume về sử dụng

```sh
mount -t glusterfs 192.168.100.50:/testvol2 /mnt
```

- Bước 3: Kiểm tra

```sh
df -hT


OUTPUT:
192.168.100.51:/testvol2 fuse.glusterfs 1014M  328M  687M  33% /mnt
```

Như vậy là tôi đã đứng trên client và mount về được volume đã tạo trên gluster server

Sau đây tôi sẽ test thử tính năng của gluster

<a name="33"></a>

### 3.3 Kiểm tra khả năng hoạt động của Gluster FS

Để kiểm tra khả năng hoạt động của gluster tôi sẽ copy 1 file iso vào thư mục `/mnt` trên client tôi đã mount volume

#### Do ở đây tôi tạo replicated volume nên theo đặc tính của của loại volume này tôi sẽ nhìn thấy được dữ liệu cả ở trên 2 server gluster

- Bước 1: Copy file iso vào thư mục `/mnt`

<img src="http://i.imgur.com/PoXCFBx.png">

- Bước 2: Kiểm tra trên server gluster01

<img src ="http://i.imgur.com/89MqQA3.png">

- Bước 3: Kiểm tra trên server gluster02

<img src="http://i.imgur.com/SRj6ue3.png">









	















