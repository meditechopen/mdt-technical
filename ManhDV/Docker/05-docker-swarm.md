## Docker Swarm

Swarm node cung cấp cấp giải pháp all-built-in Docker cluster bao gồm orchestrator của các contaienr và network cho multi host. Mục tiêu của Docker Swarm là : 
 - 1. High avaibility
 - 2. Scaling
 - 3. Load balancing
 
Một Swarm cluster được tạo bởi các node manager và các node worker :
 - **Managers** : các node chịu trách nhiệm cho việc quản lý như : quản lý cluster và các dịch vụ chạy trên nó. 
 - **Worker** : các node chạy các dịch vụ của user.
 
Ban đầu, cluster sẽ gồm 3 node :1 node manager và 2 node worker. Trong Swarm mode, 1 node có thể là manager và worker đồng thời. Với mô hình multiple manager thì số node được khuyến nghị là 3 node. Swarm manager sử dụng [Raft](https://raft.github.io/) consensus algorithm để quản lý trạng thái swarm. Raft sử dụng `quorum` cho các propose update tới swarm và đồng bộ trạng thái cho tất cả các manager node

### 1. Setup Swarm

Trên cả 3 node, cài đặt Docker engine. Các port sau phải được mở trên cluster network : 
 
 - TCP port 2377 : port liên lạc trong việc quản lý cluster
 - TCP và UDP port 7946 : liên lạc giữa các node
 - UDP port 4789 : overlay network traffic

Port 2375 và 2376 (cho TLS) nên được mở ở frontend network cho Docker API service trong trường hợp cần sử dụng việc quản lý từ xa. 

Đảm bảo Docker engine daemon chạy trên tất cả các host machine :

```sh
systemctl start docker
systemctl enable docker
systemctl status docker
```

Login tới manager node và tạo swarm 

```sh
docker swarm init --advertise-addr ens33
```

Option `--advertise-addr ens33` cho biết cluster back network được attach tới port `ens33`. Các node khác trên swarm cần phải kết nối được tới manager thông qua interface này. Mô hình này sử dụng `ens32` cho việc try cập tới các service của user và `ens33` cho các traffic clustering giữa các node.

Kiểm tra swarm node status : 

```sh
docker node list
ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
tkxuxun03da7y2bozka50mpxi *  swarm00   Ready   Active        Leader
```

Thêm các node khác vào cluster như là worker node, ta cần 1 token từ swarm manager
```sh
docker swarm join-token worker
To add a worker to this swarm, run the following command:

    docker swarm join \
    --token SWMTKN-1-3mbvdd9cay5eobj2om2pg5bdnl22z0qntmvvvslzsyt14mhgro-2qdyke72teir2erm9tuezaudj \
    192.168.2.60:2377
```

Login vào các node khác và thêm chúng vào cluster :
```sh
docker swarm join \
    --token SWMTKN-1-3mbvdd9cay5eobj2om2pg5bdnl22z0qntmvvvslzsyt14mhgro-2qdyke72teir2erm9tuezaudj \
    192.168.2.60:2377

This node joined a swarm as a worker.
```

Trên manager, kiểm tra swarm :
```sh
docker node list
ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
ickjus9bta1julsad2ls3paum    swarm01   Ready   Active
ryc6zlbxrhvblc7lk8pl3jsva    swarm02   Ready   Active
tkxuxun03da7y2bozka50mpxi *  swarm00   Ready   Active        Leader
```

Các worker node có thể thúc đẩy để trở thành có manager role:
```sh
docker node promote swarm01 swarm02

docker node list

ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
3kru4v3w3lezys0tc9cnoczou *  swarm00   Ready   Active        Leader
e2kqucdbmkzm5ks6s2pciyg9v    swarm01   Ready   Active        Reachable
mdmnfom70hh86go8up5zl272y    swarm02   Ready   Active        Reachable
```

Các node có thể được thêm như là manager bằng lấy token từ manager đã có :
```sh
docker swarm join-token manager
To add a manager to this swarm, run the following command:

    docker swarm join \
    --token SWMTKN-1-06y3xg5vjkh0tzla9fodgp6zrzsqqus8974b75umbvxeemwds9-1dlmdl3jxiz27ptggus4d9plg \
    10.10.10.60:2377
```

và chạy join command trên node mới :
```sh
[root@swarm09 ~]# docker swarm join \
    --token SWMTKN-1-06y3xg5vjkh0tzla9fodgp6zrzsqqus8974b75umbvxeemwds9-1dlmdl3jxiz27ptggus4d9plg \
    10.10.10.60:2377
```

Khi thêm các manager node tới 1 swarm, chú ý tới datacenter topology. Các distribute manager node cần ít nhất 3 avaibility zone để phòng khi toàn bộ cụm máy có vấn đề.

Để giáng cấp 1 node manager thành worker :
```sh
run docker node demote swarm02
```

Remove node ra khỏi swarm :
```sh
run docker node rm swarm01
```

Để rejoin 1 node tới swarm với trạng thái mới :
```sh
docker swarm join swarm01
```

Trong môi trường production, nên cân nhắc chạy các dịch vụ của user trên workker node để tránh việc thiết tài nguyên CPU và RAM. Để tránh việc ảnh hưỡng giữa các hành động quản lý và các service người dùng, bạn có thể `drain` 1 manager node để khiến nó không khả dụng cho các dịch vụ :
```sh
docker node update --availability drain swarm00
ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
3kru4v3w3lezys0tc9cnoczou *  swarm00   Ready   Drain         Leader
e2kqucdbmkzm5ks6s2pciyg9v    swarm01   Ready   Active        Reachable
mdmnfom70hh86go8up5zl272y    swarm02   Ready   Active        Reachable
```

Khi drain 1 node, scheduler sẽ phân bố lại các service của user chạy trên node này tới các worker node khác trên cluster.
Nếu bạn chỉ muốn sửa lỗi 1 node mà không dịch chuyển các service đã tồn tại, bạn có thể `pause` node : 
```sh
docker node update --availability pause swarm00
ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
3kru4v3w3lezys0tc9cnoczou *  swarm00   Ready   Pause         Leader
e2kqucdbmkzm5ks6s2pciyg9v    swarm01   Ready   Active        Reachable
mdmnfom70hh86go8up5zl272y    swarm02   Ready   Active        Reachable
```

Để `active` lại node :
```sh
docker node update --availability pause swarm00
ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
3kru4v3w3lezys0tc9cnoczou *  swarm00   Ready   Active        Leader
e2kqucdbmkzm5ks6s2pciyg9v    swarm01   Ready   Active        Reachable
mdmnfom70hh86go8up5zl272y    swarm02   Ready   Active        Reachable
```

### 2. Swarm networking
Swarm mode setup tạo ra một networking layout dựa trên overlay network driver. Overlay network là một network được dựng trên top của các network khác. Các node trong overlay network có thể được kết nối bởi các virutal hoặc logical link, mỗi link tương ứng với 1 phân xuyên suất của một hoặc nhiều physical link trên các network lớp dưới.

Swarm mode thúc đẩy trên các overlay network để khiến tất cả các container đang chạy trên bất kỳ node nào trên cluster cũng thuộc về cùng 1 L2 network. Một khi đã được kích hoạt, swarm tạo 1 overlay layout để kết nối tấ cả các container với nhau, không quan trọng container đang chạy trên node nào.

Trong phần này, chúng ta sẽ xem chi tiết layout này hoạt động. Swarm overlay network dựa bên VXLAN technology.

Login vào 1 trong các node các kiểm tra network :
```sh
docker network list
NETWORK ID          NAME                DRIVER              SCOPE
862ce491c4d4        bridge              bridge              local
e73fde81ef50        docker_gwbridge     bridge              local
f0c9ed46f0b6        host                host                local
1qc6vhwhaeqn        ingress             overlay             swarm
eaef890efed3        none                null                local
```

Có thể thấy over network với tên `ingress` và 1 gateway bridge network tên là `docker_gwbridge`. Kiểm tra kỹ càng với overlay đầu tiên :
```sh
docker network inspect ingress
```

```sh
[
    {
        "Name": "ingress",
        "Id": "1qc6vhwhaeqn0n9z2hdlawz72",
        "Created": "2017-03-27T15:47:53.139535526+02:00",
        "Scope": "swarm",
        "Driver": "overlay",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "10.255.0.0/16",
                    "Gateway": "10.255.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Containers": {
            "ingress-sbox": {
                "Name": "ingress-endpoint",
                "EndpointID": "c8295e304e17f569575a6c0ba35c0d41a25515672776cf2da95f3715096ba442",
                "MacAddress": "02:42:0a:ff:00:03",
                "IPv4Address": "10.255.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.driver.overlay.vxlanid_list": "4096"
        },
        "Labels": {},
        "Peers": [
            {
                "Name": "swarm00-a73dffac9ac0",
                "IP": "192.168.2.60"
            },
            {
                "Name": "swarm01-5bed1d4ed40a",
                "IP": "192.168.2.61"
            },
            {
                "Name": "swarm02-c97504fdbbf3",
                "IP": "192.168.2.62"
            }
        ]
    }
]
```

Trên network này có dải là `10.255.0.0/16`. Đây là 1 container gọi là `ingress-sbox`. Container này là container đặc biệt được tạo tự động bởi swarm trên mỗi node trong cluster.

Inspect gateway bride network 
```sh
docker network inspect docker_gwbridge
```

```sh
[
    {
        "Name": "docker_gwbridge",
        "Id": "e73fde81ef50f8bc38516ff0545be26a150af8a4020fbc4ff07b5bb5050db84f",
        "Created": "2017-03-14T16:56:46.874874148+01:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Containers": {
            "ingress-sbox": {
                "Name": "gateway_ingress-sbox",
                "EndpointID": "ee7552c5fd805e4f186eeb3d2ccf0576abdf0fca00c53f07cbb33d1a399610bc",
                "MacAddress": "02:42:ac:12:00:02",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.enable_icc": "false",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.name": "docker_gwbridge"
        },
        "Labels": {}
    }
]
```

Dải mạng này là `172.18.0.0/16`, đây là container được gọi là `gateway_ingress-sbox`. Kiểm tra network namespace :
```sh
ip netns list
1-1qc6vhwhae (id: 1)
ingress_sbox (id: 2)
```

Có 2 namespace. Kiểm tra interface trên namespace `ingress_sbox`, chúng ta sẽ thấy 2 interface :
```sh
ip netns exxec ingress_sbox ip addr
...
51: eth0@if52: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP
    link/ether 02:42:0a:ff:00:03 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.255.0.3/16 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:aff:feff:3/64 scope link
       valid_lft forever preferred_lft forever

53: eth1@if54: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP
    link/ether 02:42:ac:12:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 1
    inet 172.18.0.2/16 scope global eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::42:acff:fe12:2/64 scope link
       valid_lft forever preferred_lft forever
```

Interfce đầu tiên `eth0` trên ingress overlay network và `eth1` trên gateway bridge network. Mục tieuencuar ingress namespace sandbox là cung cấp entry point để phơi service ra bên ngoài. Chúng ta sẽ thấy nó khi triển khai 1 số service trong cluster :
```sh
ip netns exec 1-1qc6vhwhae ip addr

1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever

2: br0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP
    link/ether 76:18:82:e1:ff:26 brd ff:ff:ff:ff:ff:ff
    inet 10.255.0.1/16 scope global br0
       valid_lft forever preferred_lft forever
    inet6 fe80::c8cf:56ff:fec9:4d93/64 scope link
       valid_lft forever preferred_lft forever

50: vxlan1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue master br0 state UNKNOWN
    link/ether a2:9e:a4:4f:3b:1b brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::a09e:a4ff:fe4f:3b1b/64 scope link
       valid_lft forever preferred_lft forever

52: veth2@if51: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue master br0 state UP
    link/ether 76:18:82:e1:ff:26 brd ff:ff:ff:ff:ff:ff link-netnsid 1
    inet6 fe80::7418:82ff:fee1:ff26/64 scope link
       valid_lft forever preferred_lft forever
```
	   
	   