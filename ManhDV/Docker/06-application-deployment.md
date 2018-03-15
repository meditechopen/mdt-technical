## Application Deployment
Trong phần này chúng ta sẽ tập trung vào việc triển khai ứng dụng trên Docker Swarm. Với Swarm mode, hệ sinh thái docker trở thành 1 nền tảng hoàn chỉnh cho devops, bao gồm việc chỉ huy, thiết lập multi host networking và hỗ trợ triển khai ứng dụng.

 - Application stacks
 - Service mode
 - Placement Constrains
 - Updates Config
 - Networks
 - Volumes
 - Secrets
 
### 1. Application stacks

Một stack là 1 tập hợp các service để tạo nên một ứng dụng trên một môi trường đặc biệt. Một stack file là 1 file có dạng `yaml` định nghĩa một hoặc nhiều service và cách thức chúng liên kết với nhau. Stacks là phương thức thuận tiện để có thể tự động triển khai nhiều service mà có thể liên kết lại được với nhau, mà không cần định nghĩa chúng một cách riêng rẽ.

Các stack file định nghĩa các biến môi trường, các deployment tag, số lượng container, và các cấu hình môi trường liên quan như network và volume.

Ví dụ sau là 1 ứng dụng 2 lớp của 1 web app đang chạy và 1 mysql database là backend. 2 service được liên kết với nhau qua internal overlay network.

Sau đây là file `vote-stack,yaml` định nghĩa stack :
```sh
version: "3"
services:

  vote:
    image: docker.io/kalise/flask-vote-app:latest
    environment:
      DB_TYPE: mysql
      DB_HOST: mysql
      DB_PORT: 3306
      DB_NAME: votedb
      DB_USER: user
      DB_PASS: password
    ports:
      - "20000:5000"
    networks:
      - application_network
    volumes:
      - log-volume:/app/log:rw
    deploy:
      mode: replicated
      replicas: 1
      update_config:
        parallelism: 1
        delay: 30s
        failure_action: pause
        max_failure_ratio: 0
      restart_policy:
        condition: on-failure
      placement:
        constraints: [node.role == worker]

  mysql:
    image: mysql/mysql-server:latest
    environment:
      MYSQL_ROOT_PASSWORD: password
      #MYSQL_RANDOM_ROOT_PASSWORD: yes
      MYSQL_DATABASE: votedb
      MYSQL_USER: user
      MYSQL_PASSWORD: password
    volumes:
      - data:/var/lib/mysql:rw
    networks:
      - application_network
    deploy:
      replicas: 1
      placement:
        constraints: [node.role == worker]

volumes:
  log-volume:
    driver: local

  data:
    driver: local

networks:
  application_network:
    driver: overlay
    driver_opts:
      com.docker.network.driver.overlay.vxlanid_list: 9000
    ipam:
     driver: default
     config:
       - subnet: 172.128.1.0/24
```

Có 3 section chính trong file trên :`services`, `volumes` và `networks`

Bắt đầu với section service, hãy đi sâu vào các option :

 - `image` : image được dùng cho service
 - `port` : định nghĩa port mapping
 - `enviroment` : list các biến env được nghĩa trong Dockerfile liên quan.
 - `networks` : network mà container được attach đến
 - `volumes` : liệt kê volume được định nghĩa trong Dockerfile liên quan
 - `deploy` : định nghĩa các tiêu chuẩn, bao gồm replica, restart option, update và placement policy
 
Trên manager node, triển khai stack trên : 
```sh
[root@swarm00 ~]# docker stack deploy --compose-file vote-stack.yaml myapp
Creating network myapp_internal
Creating service myapp_mysql
Creating service myapp_vote

[root@swarm00 ~]# docker stack list
NAME   SERVICES
myapp  2

[root@swarm00 ~]# docker stack services myapp
ID            NAME          MODE        REPLICAS  IMAGE
38decu8yxubf  myapp_vote    replicated  1/1       docker.io/kalise/flask-vote-app:latest
d5dpzidowg31  myapp_mysql   replicated  1/1       mysql/mysql-server:latest

[root@swarm00 ~]# docker stack ps myapp
ID            NAME            IMAGE                                      NODE     DESIRED STATE  CURRENT STATE           
ztjwvthbckr6  myapp_proxy.1   docker.io/kalise/flask-vote-app:latest     swarm02  Running        Running 38 seconds ago
6cxjz59qgmel  myapp_nodejs.1  mysql/mysql-server:latest                  swarm01  Running        Running 40 seconds ago
```

Chúng ta có thể thấy ứng dụng tạo bởi 2 service được liên kết với nhau. Một overlay network cũng được tạo.

Inspect proxy và nodejs service :
```sh
[root@swarm00 ~]# docker service list
ID            NAME          MODE        REPLICAS  IMAGE
tvom1dh8dr3h  myapp_vote    replicated  1/1       docker.io/kalise/flask-vote-app:latest
xfme1hzygte0  myapp_mysql   replicated  1/1       mysql/mysql-server:latest

[root@swarm00 ~]# docker service inspect myapp_vote --pretty
[root@swarm00 ~]# docker service inspect myapp_mysql --pretty
```

```sh
ID:             fx6p9kx87bui2r9tw1zani806
Name:           sample_vote_app_vote
Labels:
 com.docker.stack.namespace=sample_vote_app
Service Mode:   Replicated
 Replicas:      2
Placement:Contraints:   [node.role == worker]
UpdateConfig:
 Parallelism:   1
 Delay:         30s
 On failure:    pause
 Max failure ratio: 0
ContainerSpec:
 Image:         docker.io/kalise/flask-vote-app:latest
 Env:           DB_HOST=mysql DB_NAME=votedb DB_PASS=password DB_PORT=3306 DB_TYPE=mysql DB_USER=user
Mounts:
  Target = /app/log
   Source = sample_vote_app_log-volume
   ReadOnly = false
   Type = volume
Resources:
Networks: n0axrw5wxkmk68xjclqdmqr5m
Endpoint Mode:  vip
Ports:
 PublishedPort 20000
  Protocol = tcp
  TargetPort = 5000
```

```sh
ID:             u2w47czycszmjnwrmviqr1oxb
Name:           sample_vote_app_mysql
Labels:
 com.docker.stack.namespace=sample_vote_app
Service Mode:   Replicated
 Replicas:      1
Placement:Contraints:   [node.role == worker]
ContainerSpec:
 Image:         mysql/mysql-server:latest
 Env:           MYSQL_DATABASE=votedb MYSQL_PASSWORD=password MYSQL_ROOT_PASSWORD=password MYSQL_USER=user
Mounts:
  Target = /var/lib/mysql
   Source = sample_vote_app_data
   ReadOnly = false
   Type = volume
Resources:
Networks: n0axrw5wxkmk68xjclqdmqr5m
Endpoint Mode:  vip
```

Đi sâu vào các service 

### 2. Service mode
Số replicated được định nghĩa chỉ ra số lượng container phải chạy trên bất kỳ thời điểm nào. Nếu vì 1 số lý do nào, số lượng container thấp hơn dự kiến, swarm sẽ tạo các container mới để đúng với số lượng replicated đã được set. Repricated mặc định :`mode: replicated`, các option khác là global, nghĩa là chỉ có 1 container cho mỗi node của cluster

Để start 1 global service trên mỗi node khả dụng, sử dụng `mode: global`. Mỗi khi node mới trở nên khả dụng, scheduler đặt 1 task cho global service trên node mới.

### 3. Placement constraint
Placement constraint buộc swarm phải thực hiện các kế hoạch theo tiêu chuẩn. Các service dưới đều buộc phải thực hiện trên node có role là worker bởi option `[node.role == worker]. Các option khác có thể là node name, node ID hoặc 1 vài node label. VD bạn có thể buộc swarm phải thực hiện trên node với hostname cụ thể bằng option `[node.hostname == swarm02]

### 4. Update config
Việc cập nhập cấu hình là cách để service được cập nhập. Điều này hữu dụng cho việc `rolling update` service. Trong quá trình sống của 1 ứng dụng, một số service cần được udpate. Để update 1 service mà không có thời gian gián đoạn, swarm update một hoặc nhiều container tại một thời điểm, hơn là down toàn bộ service.

VD, update vote service với image khác :
```sh
[root@swarm00 ~]# docker service update \
   --image docker.io/kalise/flask-vote-app:latest:2.4 myapp_vote
```

Swarm stop các container cũ đang chạy với lastest service và thay thế với image chỉ định. Update được tạo trên 1 container tại thời điểm đó. Các option cấu hình việc update :

 - Parallelism : cố lượng của container được update trong 1 thời điểm (trường TH này là 1)
 - Delay : time chờ đợi giữa việc update 1 group của các container (trường TH này là 30s)
 - On failure : định nghĩa hành động container thực hiện nếu việc update bị fail. Có 2 option là : continue và pause (mặc định là pause)
 - Max failure ratio : tỷ lệ thất bại có thể chịu trong quá trình update (trong TH là 0)
 
### 5. Network
Các top level network key trong stack file :
```sh
...
networks:
  application_network:
    driver: overlay
    driver_opts:
      com.docker.network.driver.overlay.vxlanid_list: 9000
    ipam:
     driver: default
     config:
       - subnet: 172.128.1.0/24
...
```

Cấu hình chính là : 

 - Driver : driver được dùng cho network, thông thường là bride hoặc overlay
 - Driver option : Phụ thuộc vào network driver
 - IPAM config : quản lý IP và các option liên quan
 - Internal mode : nếu bạn muốn tạo một overlay network bên ngoài và cô lập, không kết nối tới default gatewaty network.
 - External : khi set = true, swarm sẽ sử dụng một network đã có, thay vì tạo 1 cái mới với tên `stack-name_network_name`.
 - Labels : Thêm metadata bằng việc sử dụng một array hoặc 1 thư viện.
 
Option `external` không thể được dùng khi kết hợp với các cấu hình network khác. Swarm sẽ không cố gắng tạo network, thay vào đó sẽ sử dụng một network đã có và sẽ tạo nên 1 error nếu như không có network nào có sẵn.
```sh
...
networks:
  existing_network_name:
    external: true
```

Để tạo 1 external network bên ngoài định nghĩa stack :
```sh
[root@swarm00 ~]# docker network create \
  --driver overlay \
  --opt com.docker.network.driver.overlay.vxlanid_list=9002 \
  --subnet 172.128.0.0/24 \
  --gateway 172.128.0.1 \
  --label tenant=operation \
mynetwork
```

Mặc định, khi bạn kết nối 1 container tới 1 overlay network, Docker cũng kết nối 1 bridge network tới nó để cung cấp kết nối bên ngoài. Nếu bạn muốn tạo 1 overlay network cô lập, bạn có thể chỉ định với option `internal` :
```sh
[root@swarm00 ~]# docker network create \
  --driver overlay \
  --subnet 192.168.0.0/24 \
  --gateway 192.168.0.1 \
  --label tenant=test \
  --internal \
isolatednetwork
```

Điều này hữ dụng khi áp dụng làm backend nework cho các service.

### 6. Volumes
Top level volume key cho phép bạn định nghĩa các volume cho service. Các volume là các thư mục hoặc vùng lưu trữ bên ngoài của file system trong container, nơi container có thể sử dụng và chia sẻ dữ liệu, đồng thời dữ liệu được lưu giữ lâu dài kể cả khi container bị xóa.
```sh
...
volumes:
  log-volume:
    driver: local
...
```

volume theo form đơn giản nhất, được đặt tại filesystem của host mà container đang chạy trên đó. Mặc định, chúng nằm trong thư mục `/var/lib/docker/volumes` nếu như không chỉ định đường dẫn. Trong TH trên, nodejs service mount volume `/var/lib/docker/volumes/<stack>_log-volume` tới thư mục `/var/log` của nodejs container. Khi nodejs container bị xóa, dữ liệu vẫn còn ở tại đây. Các volume được mount dạng read-write như mặc định, nhưng cũng có thể mount với dạng read-only

Inspect volume
```sh
[
    {
        "Driver": "local",
        "Labels": {
            "com.docker.stack.namespace": "myapp"
        },
        "Mountpoint": "/var/lib/docker/volumes/myapp_log-volume/_data",
        "Name": "myapp_log-volume",
        "Options": {},
        "Scope": "local"
    }
]
```

Các option chính là : 
 
 - Driver : driver được dùng cho network, mạng định là local
 - Option : Chỉ định driver cụ thể
 - External : khi set = true, swarm sẽ dùng 1 volume đã có sẵn thay vì tạo ra 1 volume mới với tên là `stack-name_volume_name`
 
Option `external` không thể được dùng khi kết hợp với các cấu hình volume khác. Swarm sẽ không cố gắng tạo volume, thay vào đó sẽ sử dụng một volume có sẵn và sẽ tạo 1 error nếu không thấy volume có sẵn nào.

Để tạo 1 external volume bên ngoài stack :
```sh
[root@swarm00 ~]# docker volume create \
    --driver local \
    --label tenant=operations \
myvolume
```

### 7. Secrets
Trong Swarm, control plane được xác thực ra TLS và mã hóa với AES-GCM trong khi data plane không bị mã hóa bởi mặc định, vì các lý do liên quan tới hiệu năng. Swarm sử dụng `secrets` để mang lại sự an toàn cho các service. Đối với swarm, `secret` có thể là password, private key, certificate, hoặc 1 mảnh dữ liệu không nên truyền qua mạng hoặc lưu trữ mà không mã hóa trong Dockerfile hoặc source code.

Swarm cluster sử dụng secret để quản lý các dữ liệu nhạy cảm một cách tập trung và vận chuyển chúng một cách an toàn tới chỉ các container cần truy cập. Secret được mã hóa trong quá trình vận chuyển và ở phần còn lại trên swarm. 1 secret chỉ có thể truy cập được đối với các dịch vụ đã được cấp quyền truy cập đến nó, và chỉ trong khi các service task đó đang chạy.

 - usernames và passwords
 - TLS certificates và keys
 - SSH keys
 - Database names
 - Generic data up to 500 GB
 
**Chú ý** : các secret chỉ khả dụng với các dịch vụ swarm, không khả dụng với các container chạy standalone.

Tạo 1 secret password bằng việc chỉ định 1 cái tên cho secret :
```sh
echo Th1s1sA5tr0nGpa55w0rD! | docker secret create password -
blfkkqkwxbozxi6p9o49zi2i6
```

Nếu 1 secret được giữ trong 1 file, bạn có thể làm như sau : 
```sh
echo Th1s1sA5tr0nGpa55w0rD! > password.txt
docker secret create password password.txt
```

List các secret
```sh
docker secret list
ID                          NAME                CREATED             UPDATED
blfkkqkwxbozxi6p9o49zi2i6   password            9 seconds ago       9 seconds ago
```

inspect một secret password :
```sh
ID:                     blfkkqkwxbozxi6p9o49zi2i6
Name:                   password
Created at:             2017-08-09 07:28:10.556973156 +0000 utc
Updated at:             2017-08-09 07:28:10.556973156 +0000 utc
```

Secret được mã hóa hóa và phân tán tới tất cả các managern node trên swarm như key/value, đảm bảo rằng có thể HA được cho secret.

Secret phải được attach tới 1 service :
```sh
docker service create --name nginx \
                      --secret password \
                      --publish 80:80 \
                        nginx:latest
```

Secret được attach trên `/run/secrets` là in-memory file system.
Xác định container và login để kiểm tra : 
```sh
docker service ps nginx

ID             NAME       IMAGE          NODE         DESIRED STATE
qa5z7m19gy2a   nginx.1    nginx:latest   clastix00    Running           

docker exec -it nginx.1 sh
# cat /run/secrets/password
Th1s1sA5tr0nGpa55w0rD!
```

Secret có thể được map tới container với tên khác nhau :
```sh
docker service create --name nginx \
                      --secret source=password,target=/etc/nginx/password.txt \
                      --publish 80:80 \
                        nginx:latest

docker exec -it nginx.1 sh
# cat /etc/nginx/password.txt
Th1s1sA5tr0nGpa55w0rD!
```

Secret có thể bị remove và add tới các service đang chạy. Tuy nhiên, việc thêm và xóa sẽ khiến service phải redeploy.
```sh
docker service update nginx --secret-rm password
docker service update nginx --secret-add password2
```

Hoặc với command duy nhất
```sh
docker service update nginx --secret-rm password --secret-add password2
```

### 8. Secret cho sharing keys
Chúng ta sẽ sử dụng secret cho sharing TLS key và certificate với các container mà không cần đẩy chúng vào dockerfile. Image `kalise/lighttps` là 1 Lighttp web server có hỗ trợ mở TLS.

Image yêu cầu 1 PEM file `/etc/lighttpd/ssl/server.pem` bao gồm cả server key và certificate để chạy HTTPS. Code có thể tìm ở [đây](https://github.com/kalise/lighttps)

Giả dụ chúng ta đã có sẵn private key `host.key` và certificate `host.crt`. Theo yêu cầu của Lighttpd, tổ hợp chúng thành 1 file PEM duy nhất :
```sh
cat host.key host.crt > server.pem
```

Đảm bảo rằng cũng có 1 Certification Authority certificate `ca.pem` cho việc kiểm tra service.

Tạo secret 
```sh
docker secret create pemfile server.pem
rgb28ueelrtlp6whkb2c9ezn4
```

Tạo Lighttpd service với TLS đã được bật bằng việc attach các secret trên target location
```sh
docker service create  \
       --name lighttps  \
       --secret source=pemfile,target=/etc/lighttpd/ssl/server.pem \
       --publish 80:80 \
       --publish 443:443 \
       kalise/lighttps:latest
```

Kiểm tra service đang chạy :
```sh
docker service list
ID            NAME      MODE        REPLICAS  IMAGE                    PORTS
ksjygrpk012s  lighttps  replicated  1/1       kalise/lighttps:latest  *:80->80/tcp,*:443->443/tcp
```

Kiểm tra HTTPS đang hoạt động
```sh
curl --cacert ca.pem https://<hostname>:443

<html>
  <header>
  </header>
  <body>
       <h1>It Works!</h1>
  </body>
</html>
```

### 9. Secret cho việc chia sẻ pasword
Như vd về việc tạo password với secret, hãy cân nhắc với service `vote app` chúng ta đã dùng trước đó. Yaml stack file có thể tăng cường bằng việc thêm 2 secret :
```sh
secrets:
   db_password:
     file: db_password.txt
   db_root_password:
     file: db_root_password.txt
```

Secret thứ nhất sẽ dùng cho MySQL app db password như yêu cầu bởi MySQL và Vote container. Secret thứ 2 sẽ được dùng bởi chỉ container MySQL
```sh
...
  vote:
    image: docker.io/kalise/flask-vote-app:latest
    environment:
      DB_TYPE: mysql
      DB_HOST: mysql
      DB_PORT: 3306
      DB_NAME: votedb
      DB_USER: user
      DB_PASS: /run/secrets/db_password
...
  mysql:
    image: mysql/mysql-server:latest
    environment:
      MYSQL_ROOT_PASSWORD: /run/secrets/db_root_password
      #MYSQL_RANDOM_ROOT_PASSWORD: yes
      MYSQL_DATABASE: votedb
      MYSQL_USER: user
      MYSQL_PASSWORD: /run/secrets/db_password
```

Như chúng ta có thể thấy, không cần thiết phải code bất kỳ password nào trong stack definition. Để truyền password tới container nó chỉ yêu cầu tạo password với dạng text file và start các app.
```sh
echo Th1s1sA5tr0nGpa55w0rD! > db_password.txt
echo Th1s1sA5tr0nG%ooTpa55w0rD! > db_root_password.txt

docker stack deploy -c vote-stack-secrets.yaml myapp
```

Một khi start lên, swarm sẽ mã hóa password này và phân tán 1 cách an toàn tới tất cả các manager node. Các container đang chạy sẽ access password này một cách an toàn với cơ chế mã hóa mạnh mẽ được cung cấp bởi swarm.

Các container sẽ dùng secret từ secret location `run/secrets` để sử dụng các env variable. User có thể remove textfile chứa password :
```sh
rm -rf db_password.txt
rm -rf db_root_password.txt
```

Bạn phải update stack file lên bản 3.1 để có thể hỗ trợ secret. Việc update stack file có thể tìm tại [đây](http://tutorial.docker.noverit.com/examples/vote-stack-secrets.yaml)