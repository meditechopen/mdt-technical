# Fibre Channel SAN phần III: Redundancy và Multipathing

Bài viết này là phần cuối cùng của series bài về Fibre Channel. Xem các bài viết trước theo link dưới đây

[Phần I: Địa chỉ FCP và WWPN](https://github.com/trimq/mdt-technical/blob/master/TRIMQ/Storage/docs/FibreChannel-Part-I.md)

[Phần II: ZONING, LUN MASKING và FABRIC LOGIN](https://github.com/trimq/mdt-technical/blob/master/TRIMQ/Storage/docs/FibreChannel-Part-II.md)

## Mục lục

- [1. Fibre Channel Redundancy](#1)




-----------------------------------------------

<a name="1"></a>

### 1. Fibre Channel Redundancy

Các máy chủ truy cập được vào hệ thống lưu trữ sẽ luôn luôn là nhiệm vụ rất quan trọng của doanh nghiệp. Bởi vậy, chúng ta không muốn hỏng hóc ở bất cứ điểm nào. Do vậy Redundant Fibre Channel được đưa ra. Chúng ta hiểu như là hệ thống sẽ có Fabric A và Fabric B hoặc là SAN A và SAN B. Mỗi server và các hệ thống lưu trữ nên được kết nối với cả 2 fabrics với các ports HBA dự phòng.

Các switch phân tán của Fibre Channel sẽ chia sẻ thông tin với nhau (Domain ID, cở sở dữ liệu FCNS và zoning). Khi chúng ta cấu hình zoning trong một fabric, chỉ cần cấu hình trên một Switch, nó sẽ tự động phân phối đến các Switch khác. Điều này là rất tiện lợi nhưng cũng có một vài nhược điểm ở đây. Bởi vì khi ta cấu hình sai thì nó sẽ nhân bản ra các switch khác trong fabric. Nếu một lỗi xảy ra trong fabric A được truyền sang fabric B, nó sẽ làm cho cả 2 fabric bị DOWN và sẽ mất kết nối của series vào máy chủ lưu trữ. Điều này rất nguy hiểm.

Vì lí do như trên, Switch ở các side khác nhau trong fabric không được kết nối chéo với nhau. Cả 2 side của fabric đều được giữ riêng biệt. Điều này khác với cách làm việc của Ethernet, nơi mà chúng ta thường kết nối các Switch lại với nhau.

