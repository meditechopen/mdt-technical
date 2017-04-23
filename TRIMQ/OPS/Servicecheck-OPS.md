# Kiểm tra dịch vụ theo trang https://status.meditech.vn

#Dịch vụ : Openstack, HAProxy, Pacemaker, Rabbitmq_cluster


1. Kiểm tra trạng thái service Openstack : 
1.1 Keystone
## Trên node CTL
### HTTP status
systemctl status httpd

###port 5000 và 35357 trạng thái LISTEN

netstat -nltp | egrep '5000|35357'

###Check lấy token
source admin-openrc
openstack token issue


1.2 Neutron

##Trên node CTL

### Neutron server status
systemctl status neutron-server

### Neutron agent status
systemctl status neutron-metadata-agent
systemctl status neutron-dhcp-agent status		//không dùng với mô hình provider network
systemctl status neutron-l3-agent status 
systemctl status neutron-openvswitch-agent status

### Kiểm tra trạng thái neutron-agent : 

source admin-openrc
neutron agent-list

Trạng thái như sau là OK, dòng alive: :-) và admin_state_up: true: http://prntscr.com/eyjv7a . Host nào xuất hiện alive: xxx là đang NOT-OK

## Trên node COM

###Neutron openvswitch status
systemctl status neutron-openvswitch-agent status


1.3 Nova

## Trên node CTL

systemctl status nova-api
systemctl status nova-conductor 
systemctl status nova-consoleauth 
systemctl status nova-novncproxy 
systemctl status nova-scheduler 

source admic-openrc
nova service-list

Trạng thái như sau là OK : http://prntscr.com/eyjwqt

## Trên node COM
systemctl status nova-compute


1.4 Glance

## Trên node CTL
systemctl status glance-api 
systemctl status glance-registry 


1.5 Cinder

## Trên node CTL
systemctl status cinder-api 
systemctl status cinder-scheduler 

## Trên node Cinder
systemctl status cinder-volume 
systemctl status cinder-backup 


1.6 Horizon

## Trên node CTL
### HTTP status
systemctl status httpd

### Port 80/443 trạng thái LISTEN (hoặc check port riêng của Horizon sử dụng)
netstat -nltp | egrep ':80|:443'

######Cú pháp#######
Xem ảnh sau : http://prntscr.com/eyjxuc

[Telco_M][OpenStack]OpenStack Status 
Kiểm tra trạng thái các dịch vụ trong OpenStack :ok_hand:

 - Keystone *OK* 
 - Neutron *OK* 
 - Nova *OK* 
 - Cinder *OK* 
 - Glance *OK* 

_Meditech Team_ :sunglasses:


2. Kiểm tra trạng thái Pacemaker

## Trên node CTL
###Câu lệnh 
crm_mon -1

Kết quả như ảnh sau là OK : http://prntscr.com/eyjy4r


3. Kiểm tra trạng thái HAProxy

## Trên node CTL

Kiểm tra trên trang : http://10.3.10.6:8080/stats
Vào được trang và check các biểu đồ được là OK.


4. Kiểm tra trạng thái RabbitMQ-Cluster

## Trên node CTL

### Trạng thái dịch vụ RabbitMQ
systemctl status rabbitmq-server

### Trạng thái RabbitMQ_Cluster
rabbitmqctl cluster_status

Trạng thái như ảnh sau là  OK : http://prntscr.com/eyjz06


#####Cú pháp#####
[Telco_M][Service-Common]Service-Common Status 
Kiểm tra trạng thái các dịch vụ chung :ok_hand:

 - Pacemaker *OK* 
 - HAProxy *OK* 
 - RabbitMQ_Cluster *OK* 
 
_Meditech Team_ :sunglasses:

######Quy ước#######
 -	Đối với MBF : [Telco_M]]
 -	Đối với CUC : [GOV_BYT]
