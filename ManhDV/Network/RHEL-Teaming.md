##I. Network teaming trên RHEL
###1. Hiểu về network teaming
Việc kết hợp hay gộp chung các đường mạng để nhằm cung cáo một đường logic với mục đích thu được **network throughput**(1) cao hơn, hoặc cung cấp một **redundancy**(2), được biết với nhiều tên như "channel bonding", "Ethernet bonding", "port trunking", "channel teaming", "NIC teaming", "link aggregation"... Các định nghĩa này được hiểu chung trong Linux là "bonding". Thuật ngữ "Network Teaming" được coi là một cách biểu diễn mới, đưa ra như một sự lựa chọn và không thay thế bonding. Các driver bonding đã tồn tại sẽ không bị ảnh hưởng.

Network teaming (Team), được cung cấp một driver kernel nhỏ để kiểm soát nhanh các packet flow, và các ứng dụng user-space đa dạng. Driver có một API (Tean Netlink API), biểu diễu các giao tiếp Netlink. Các ứng dụng user-space có thể dùng API này để giao tiếp với driver. Một thư viện (lib), được cung cấp để user-space bao phủ các giao tiếp Team Netlink và RT Netlink messages. Một ứng dụng nền **teamd**, sử dụng Libteam **lib**. Một instance của **teamd** có thể kiểm soát một instance của Team driver. **Runner**, tiến trình nền phụ trách việc load-balancing và active-backup logic như là round-robin. 

**Teamdctl** cung cấp công cụ để kiểm soát instance của teamd đang sử dụng D-bus.

Team Netlink API giao tiếp với các ứng dụng user-space sử dụng Netlink messages. Thư viện user-space **libteam** không tương tác trực tiếp với API, mà dùng libnl hoặc teamnl để tương tác với driver API.

Tóm lại, các instance của Team driver, đang chạy trong kernel, không được cấu hình hoặc kiểm soát trực tiếp. Tất cả cấu hình được hoàn thành với sự giúp đỡ của các ứng dụng user-space, như là **teamd**. Ứng dụng sau đó sẽ tương tác trực tấp với một phần kernel driver.

####2. Hiểu về trạng thái mặc định của các interface Master và Slave.

Khi kiểm soát các teamd port interface sử dụng tiến trình NetworkManager, đặc biệt là trong quá trình tìm lỗi, hãy lưu ý các điều sau : 

 - **1**. Khởi động interface master sẽ không tự động khởi động port interface.
 - **2**. Việc start port interface luôn luôn dẫn tới việc start interface master
 - **3**. Việc stop master interface cũng dẫn tới việc stop port interface.
 - **4**. Một master không có port có thể start các kết nối IP tĩnh.
 - **5**. Một master không có port sẽ chờ các port khi khởi động DHCP connection.
 - **6**. Một master với DHCP connection sẽ chờ các port hoàn chỉnh khi một port với một carrier được thêm vào.
 - **7**. Một master với DHCP connection sẽ chờ các port sẽ tiếp tục chờ khi một port không có carrier được thêm vào.
 
**Chú ý** : Việc dùng các kết nối cap trực tiếp mà không dùng switch sẽ không được hỗ trợ bởi teaming.

####3. So sánh Network Teaming với Bonding

|Tính năng|Bonding|Team|
|---------|-------|----|
|broadcast Tx policy|Yes|Yes|
|round-robin Tx policy	|Yes|Yes|
|active-backup Tx policy|Yes|Yes|
|LACP (802.3ad) support	|Yes (passive only)|Yes|
|Hash-based Tx policy	|Yes|Yes|
|User can set hash function	|No|Yes|
|Tx load-balancing support (TLB)|Yes|Yes|
|LACP hash port select	|Yes|Yes|
|load-balancing for LACP support||No	|Yes|
|Ethtool link monitoring|Yes|Yes|
|ARP link monitoring|Yes|Yes|
|NS/NA (IPv6) link monitoring|No	|Yes|
|ports up/down delays	|Yes|Yes|
|port priorities and stickiness (“primary” option enhancement)|No	|Yes|
|separate per-port link monitoring setup|No	|Yes|
|multiple link monitoring setup	|Limited|Yes|
|lockless Tx/Rx path	|No (rwlock)|Yes (RCU)|
|VLAN support	|Yes	|Yes|
|user-space runtime control	|Limited|Full|
|Logic in user-space	|No	|Yes|
|Extensibility	|Hard	|Easy|
|Modular design	|No	|Yes|
|Performance overhead|Low|Very Low|
|D-Bus interface|No	|Yes|
|multiple device stacking|Yes|Yes|
|zero config using LLDP	|No	|(in planning)|
|NetworkManager|support|Yes|Yes|