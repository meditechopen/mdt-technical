# Fibre Channel SAN Phần 1 - Địa chỉ FCP và WWPN

Cách tốt nhất để bắt đầu với SAN  (Storage Area Networks) là giải thích Fibre Channel, giao thức ban đầu của SAN. Fibre Channel vẫn là một giao thức phổ biến trong SAN hiện nay. Mặc dù Fibre Channel trên Ethernet thu hút được nhiều sự quan tâm trong những năm tới đây


## Thuật ngữ SAN

- LUN được hiểu là  Logical Unit Number. LUN đại diện cho một logical disk tương ứng với một host. Các Clients sẽ kết nối tới LUN và sử dụng như là local disk của client. 

- Client được gọi là Initiator

- Hệ thống lưu trữ được gọi là Target


## Giao thức Fibre Channel

Giao thức Fibre Channel được sử dụng để truyền câu lệnh SCSI trên mạng Fibre Channel. Với ổ cứng local, các câu lệnh SCSI sẽ được truyền tới ổ cứng đó. Còn với SAN, câu lệnh sẽ được truyền tới ổ cứng nhưng thông qua mạng.

## Fibre Channel là Lossless

Fibre Channel là một giao thức rất ổn định và đáng tin cậy đó là một trong những lý do chính nó vẫn còn rất phổ biến với các kỹ sư storage. 

Mạng Ethernet đang bị mất mát khi truyền dữ liệu. Với TCP, người gửi sẽ gửi lưu lượng đến người nhận và người nhận sẽ gửi xác nhận lại. Nếu người gửi không nhận được thông tin phản hồi, nghĩa là dữ liệu đã bị mất mát trong quá trình truyền đi và người gửi sẽ gửi lại.

UDP không cần xác nhận lại từ phía người nhận. Tùy thuộc vào các lớp ứng dụng cao hơn để đối phó với bất kỳ lưu lượng truy cập bị mất

Fibre Channel không bị mất mát, không giống như TCP và UDP. Cơ chế kiểm soát lưu lượng buffer to buffer được tích hợp sẵn trong giao thức để đảm bảo các frame không bị mất.


## Tốc độ của Fibre Channel

Fibre Channel hiện hỗ trợ băng thông từ 2, 4, 6, 8 và 16 Gbps. Không phải tất cả phần cứng đều hỗ trợ tốc độ cao hơn. Do đó, tốc độ bạn sẽ nhận được phụ thuộc vào thiết bị bạn đã triển khai

## Mạng Fibre Channel

Fibre Channel khác với Ethernet ở tất cả các tầng mô hình OSI, bao gồm cả tầng Physical. Do đó, nó đòi hỏi phải có card mạng, dây cáp và các switch chuyên biệt. Bạn không thể sử dụng một card mạng Ethernet hoặc một switch Ethernet cho Fibre Channel. Điều này khác với các giao thức iSCSI, FCoE và NAS của chạy qua Ethernet.

FCoE là 1 trường hợp đặc biệt sử dụng Ethernet nhưng vẫn chạy được Fibre Channel, yêu cầu lưu lượng mạng không được mất mát. Do đó nó không thể sử dụng card mạng Ethernet và Switch Ethernet. Điều này sẽ được đề cập ở blog sau. 

Trong sơ đồ sau đây, máy chủ web nằm ở giữa và máy client nằm ở trên cùng. Client sẽ truy cập vào trang web của máy chủ web server. Máy client sẽ truy cập vào máy chủ qua mạng Ethernet cục bộ thông thường. Sau đó, để lấy thông tin trang web. Sau đó máy chủ sẽ kết nối với bộ lưu trữ của nó thông qua mạng Fibre Channel.

<img src="http://www.flackbox.com/wp-content/uploads/2016/07/Fibre-Channel-01-768x364.jpg">

Với mạng local kết nối với máy client, sử dụng chuẩn Ethernet. Thường thì sẽ sử dụng 2 cổng để đảm bảo trên cùng 1 card vật lý hoặc các card riêng rẽ.

Đối với mạng kết nối tới Storage, máy chủ phải sử dụng cổng HBA (Host Bus Adapter). Mỗi HBA là một Fibre Channel ứng với một Ethernet port.


## Network Addressing – The WWN

Fibre Channel sử dụng WWN (World Wide Names) để gán địa chỉ mỗi Initiator và Target đều được gán WWN. WWN là địa chỉ 8 byte được tạo bởi 16 kí tự hexadecimal. Ví dụ:

```sh
21:00:00:e0:8b:05:05:04
```

Có hai loại địa chỉ WWN: WWNN (World Wide Node Name) và WWPN (WWPN World Wide Port Name). Cả hai đều sử dụng cùng một định dạng và giống nhau.

### WWNN (World Wide Node Name)

WWNN (World Wide Node Name) được gán cho một host trong mạng lưu trữ. WWNN có biểu thị cho riêng 1 host. Cùng một WWNN có thể xác định nhiều interface của một network node đơn. Một host có thẻ có nhiều HBA hoặc nhiều port trong 1 HBA.

Đôi khi bạn có thể nhìn thấy WWNN cũng được tham chiếu như NWWN (Node World Wide Name). WWNN và NWWN là chính xác là một, chỉ có hai cách nói khác nhau.

### WWPN (World Wide Port Name)

Mỗi host cũng có World Wide Port Names. Khác biệt của WWPN là được gán cho tất cả các port trên host. Nếu chúng ta có một HBA nhiều port trong cùng một máy chủ lưu trữ. Mỗi port trên đó HBA sẽ có một WWPN khác nhau. WWPNs tương đương với địa chỉ MAC trong Ethernet. WWPN là địa chỉ cứng của nhà sản xuất của HBA. Và nó được bảo đảm là duy nhất trên toàn cầu.

Cũng giống như WWNN, WWPN cũng được gọi là PWWN. 

Cả Initiator và Target đều được gán WWNN và WWPN trong mạng Fibre Channel để giao tiếp với nhau.

### Aliases

WWPN là một địa chỉ hexadecimal lớn và dài. Do vậy rất khó khi theo dõi và khó để nhập vào khi cấu hình. Các Aliases có thể được cấu hình cho WWPN để cấu hình và xử lý sự cố dễ dàng hơn. 






