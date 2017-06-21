# Fibre Channel SAN phần II: ZONING, LUN MASKING và FABRIC LOGIN

## Mục lục:
- [1. Zoning](#1)
- [2. LUN Masking](#2)
- [3. Switch Domain IDs](#3)
- [4. FLOGI](#4)
- [5. The Fibre Channel Name Service](#5)
- [6. Port Login](#6)
- [7. Quá trình LOGIN](#7)
- [8. Tài liệu gốc](#8)

-------------------------------------------------

### 1. Zoning

Để bảo mật, zoning được cấu hình trên Fibre Channel switch để điều khiển Fibre Channel port có khả năng giai tiếp với nhau. Chúng ta cho phép port trên client (Initiator) giao tiếp với port của hệ thống lưu trữ (Target). Các Initiator không được giao tiếp với nhau qua mạng Fibre Channel, điều này làm tăng tính bảo mật và giảm lưu lượng truy cập, giúp cho mạng Fibre Channel ổn định và đáng tin cậy.


Các nhà sản xuất switch Fibre Channel phổ biến là Cisco và Brocade. Ví dụ dưới đây là dành cho Switch Cisco nhưng cấu hình của Brocade cũng tương tự.

Trong ví dụ mô hình dưới đây, có 2 máy chủ đóng vai trò client (Initiator) đang trong trạng thái DOWN. Ở dưới cùng trong hình. 1 máy chủ đóng vai trò storage server trong trạng thái UP. Các aliases được gán map với các WWPN. Đã cấu hình aliases trên switch cho các WWPN của server và hệ thống storage 

<img src="http://www.flackbox.com/wp-content/uploads/2016/07/Fibre-Channel-02-1-768x428.jpg">

Các zones riêng biệt được cấu hình cho các yêu cầu kết nối riêng biệt. Cấu hình cho 1 zone (tên là SERVER1) để Server 1 giao tiếp được với storage system. Trong zone SERVER1 bao gồm các member là fcalias SERVER1 và fcalias NETAPP-CTRL1

Trong 1 zone riêng biệt khác (tên là SERVER2) để Server 2 kết nối đến storage system. Gồm fcalias SERVER2 và fcalias NETAPP-CTRL1

Sau đó group các zones đó lại bên cạnh nhau và thực hiện trên switch. Đặt tên cho zone đó là MY-ZONESET. Zone này sẽ bao gồm SERVER1 và SERVER2.

Trong mô hình này, 2 máy chủ client là Server 1 và Server 2 có thể giao tiếp với máy chủ Storage nhưng không thể giao tiếp lẫn nhau vì chúng không thuộc cùng zone. 

Đây là một ví dụ đơn giản để hiểu được zone hoạt động như nào. Thông thường, chúng ta sẽ có ít nhất 2 máy chủ storage và các switches để đảm bảo tính dự phòng. Hãy xem phần III trong loạt bài về Fibre Channel này để hiểu trên thực tế hệ thống được triển khai như nào.

### 2. LUN Masking

Cũng giống như việc cấu hình zone phải thực hiện trên switch. Chúng ta cấu hình LUN Masking trên hệ thống lưu trữ. Điều quan trọng là LUN được thể hiện cho đúng host. Nếu sai host, khi kết nối đến sẽ bị hỏng hóc.

Zone trên các switches đảm bảo rằng các server không thể giao tiếp được với nhau, nhưng có thể giao tiếp với hệ thống lưu trữ. Vậy làm sao để các server đó không kết nối vào LUN của nhau, đó là lí do có LUN Masking.

Zone trên các thiết bị switches ngăn không cho các máy chủ truy cập trái phép đến được hệ thống lưu trữ và ngăn không cho các máy chủ giao tiếp với nhau qua mạng fibre channel. Nhưng nó không ngăn cản một máy chủ truy cập sai LUN một khi nó đã được vào hệ thống lưu trữ. LUN Masking được cấu hình trên hệ thống lưu trữ để khóa một LUN xuống đến máy chủ hoặc máy chủ được ủy quyền để truy cập đến nó. Để đảm bảo tính an toàn cho việc lưu trữ của bạn, bạn cần phải cấu hình zone trên switch và LUN Masking trên hệ thống lưu trữ.

Dưới đây là một ví dụ về cấu hình LUN Masking. Server1 và Server2 là các máy chủ không có disk. Tôi cấu hình Boot các LUN cho cả hai Server1 và Server2. Đối với Server1 Boot LUN, chỉ duy nhất initiator đó có thể kết nối tới đó là WWPN của Server1. Tương tự với Server2. Điều này ngăn cản máy chủ kết nối sai và có khả năng làm hư hỏng LUN của máy chủ khác. Ở đây cũng có thể sử dụng aliases thay vì WWPN.

<img src="http://www.flackbox.com/wp-content/uploads/2016/07/Fibre-Channel-03-768x412.jpg">

### 3. Switch Domain IDs

Tiếp theo là Switch Domain IDs. Mỗi Switch trong mạng Fibre Channel sẽ được gán Domain ID riêng. Cái tên này có thể gây bối rối, vì bạn sẽ nghĩ Domain ID sẽ đại diện cho toàn bộ tên miền của các Switches. Nhưng nó không có nghĩa cho tất cả như vậy. Domain ID thực chất chỉ là một ID duy nhất cho Switch bên trong mạng Fibre Channel. Domain ID là một giá trị từ 1 đến 239 ở cả Switch Cisco và Brocade.

Một Switch trong mạng sẽ được tự động gán là Principle Switch. Nó có trách nhiệm đảm bảo mỗi Switch bên trong mạng có một Domain ID duy nhất.

Mỗi Switch sẽ học từ Switch khác trong mạng và làm sao để định tuyến dựa vào Domain ID.


## Tiến trình login

### 4. FLOGI

Khi một Server hoặc port HBA của hệ thống lưu trữ được bật lên. Nó sẽ gửi yêu cầu FLOGI (Fabric Login request) sẽ bao gồm WWPN. Để Fibre Channel Switch cắm trực tiếp vào. Switch sau đó sẽ chỉ định gán 24-bit FCID vào đó. Đó là Fibre Channel ID Address. Nói 1 cách đơn giản, host sẽ nói "Đây là WWPN của tôi, hãy gán cho tôi một FCID để tôi có thể giao tiếp trong mạng Fibre Channel"

FCID được gán cho host được tạo thành từ Domain ID của switch và switch port được cắm vào. FCID tương tự như địa chỉ IP trong Ethernet. Nó được sử dụng bởi các thiết bị switch Fibre Channel để định tuyến giữa các server và hệ thống lưu trữ của các server đó. Switch duy trì một bảng các FCID để ánh xạ địa chỉ WWPN và những gì port host nằm lên đó.

Các Switches Fibre Channel chia sẻ thông tin lẫn nhau. Mọi switches trong mạng của bạn sẽ học Domain ID của tất cả các thiết bị switches khác. Nó cũng học về WWPN của tất cả các host được gắn vào mạng và FCID được ánh xạ tới các WWPN. Dựa trên FCID, nó biết Domain ID của switch mà mỗi host được cắm vào (vì phần đầu tiên của FCID là Domain ID). Bởi vì họ có tất cả thông tin này, họ có thể chuyển đổi lưu lượng truy cập giữa các máy chủ.

Dưới đây là một sơ đồ của quá trình đăng nhập vải làm việc. Khi Server1 được bật lên, cổng HBA sẽ gửi một FLOGI tới switch mà nó được gắn vào. Switch sẽ được gán vào đó một FCID. Nếu bây giờ tôi chạy lệnh show cơ sở dữ liệu FLOGI trên switch đó, tôi sẽ thấy interface mà host được cắm vào, FCID, WWPN của nó. Trong ví dụ ở đây tôi đã cấu hình aliase SERVER1 cho WWPN.

<img src="http://www.flackbox.com/wp-content/uploads/2016/07/Fibre-Channel-04-768x415.jpg">

Điều tương tự xảy ra với hệ thống lưu trữ của chúng tôi ở phía trên cùng của sơ đồ. Khi nó bật, cổng HBA sẽ gửi FLOGI tới switch mà nó được cắm vào. Switch sẽ gán cho port đó một FCID. Và nếu show cơ sở dữ liệu trên Switch, ta sẽ thấy interface mà storage system được cắm vào, đó là FCID, WWPN và aliase đã được cấu hình (ở ví dụ này là NETAPP-CTRL1).

### 5. The Fibre Channel Name Service

Các switches fibre channel chia sẻ thông tin về cơ sở dữ liệu FLOGI lẫn nhau sử dụng FCNS (Fibre Channel Name Service). Mỗi switch trong mạng sẽ học WWPN, và làm sao để định tuyến.

Lệnh show cơ sở dữ liệu FLOGI trên các thiết bị switches của Cisco sẽ chỉ hoạt động trên switch nơi mà các máy client được cắm trực tiếp vào. Các thiết bị switch chia sẻ thông tin cục bộ với nhau thông qua FCNS. Nếu chúng ta làm một cơ sở dữ liệu FCNS hiển thị trên một switch Cisco, chúng ta sẽ thấy được các FCID và WWPN của các host trong mạng. Bởi vì FCID có bắt nguồn từ Domain ID. Đó là cách xác định thiết bị switch của, các thiết bị switch giờ đây biết cách định tuyến lưu lượng tới bất kỳ máy chủ lưu trữ nào trong mạng.

<img src="http://www.flackbox.com/wp-content/uploads/2016/07/Fibre-Channel-05-768x443.jpg">


### 6. Port Login

Sau khi tiến trình FLOGI Fabric Login được hoàn tất. Initiator sẽ gửi một Port Login (PLOGI). Dựa vào cấu hình Zone trên Switch. Server sẽ học các WWPN khả dụng trong hệ thống lưu trữ. 

Trong ví dụ dưới đây. Server1 sẽ gửi một FLOGI và sẽ được chỉ định FCID. Khi tiến trình đó được hoàn thành, nó sẽ gửi một PLOGI tới Switch local. Switch sẽ check cấu hình zone và cho phép nó nói chuyện vói storage của Server1.

<img src="http://www.flackbox.com/wp-content/uploads/2016/07/Fibre-Channel-06-768x483.jpg">

### 7. Quá trình LOGIN

Cuối cùng là PLRI (Process Login). Host Initiator sẽ gửi một yêu cầu PLRI Process Login tới server lưu trữ của nó. Hệ thống lưu trữ sẽ chấp nhận yêu cầu của host tới LUN dựa trên cấu hình LUN Masking.

<img src="http://www.flackbox.com/wp-content/uploads/2016/07/Fibre-Channel-07-768x489.jpg">


### 8. Tài liệu gốc

http://www.flackbox.com/fibre-channel-san-part-2-zoning-lun-masking-flogi-plri/


