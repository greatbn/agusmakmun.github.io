---
layout: post
title: What's a Decentralized Application?
image: "/content/images/2017/09/what-is-the-blockchain-and-why-is-it-so-important.jpg"
date: '2017-09-10 17:41:33'
tags:
- bitcoin
- blockchain
- decentralized-applications
- cryptocurrency
- tien-ao
---

#### Mở đầu
Thời gian gần đây Bitcoin nói riêng và Blockchain nói chung đang làm một trong những trend nổi tuơng đuơng với AI hiện nay. 
Hòa cùng sự phát triển đó, mình cũng muốn tìm hiểu đôi chút về những công nghệ này. 
Các bài viết thuộc series này do mình vừa đọc và dịch từ cuốn `Decentralized Applications Harnessing Bitcoin’s Blockchain Technology` của tác giả `Siraj Raval`.

Bài post này mình viết là ở Chuơng 1. Nói về việc định nghĩa một ứng dụng `Decentralized` và Bitcoin. 

##### Bitcoin là gì?

Trước khi đi vào chi tiết Decentralized Applications - từ đây xin gọi tắt là `dapps`. Ta nói về Bitcoin và Web trước. Chúng ta đã thấy các dịch vụ Web phát triển đáng kể. Hàng tỉ người trên thế giới được kết nối Internet. Về cơ bản có một chuẩn kết nối trong bộ giao thức TCP/IP.

- Link layer truyền dữ liệu trên các dây điện.
- Internet layer làm nhiệm vụ định tuyến.
- Transport layer giúp vận chuyển dữ liệu 
- Application Layer trừu tượng hóa dữ liệu trong hình thức của ứng dụng. 

Tất cả các giao thức làm việc cùng nhau để trao đổi dữ liệu, nhưng không giao đổi giá trị (value). Bitcoin hoạt động như một lớp giao thức thứ 5, cho việc trao đổi giá trị, nằm trên 4 lớp giao thức kia.

Chúng ta đang có một cách chuyển tiền thông qua Web. Vấn đề là toàn bộ việc kém hiệu quả liên quan tới các hệ thống cũ giống "Automated Clearing House (ACH)", chúng được thiết kế trước khi Internet xuất hiện. Các hệ thống thanh toán thường chậm bởi vì chúng phải yêu cầu một "Centralize clearing House". Bitcoin giúp giải quyết vấn đề này bằng cách: Các máy tính không phải đợi hàng ngày cho một hóa đơn để "clear", chúng có thể giao tiếp trực tiếp với một máy khác. Chúng có thể gửi hàng tỉ các hóa đơn nhỏ lẻ cho nhau mà không phải mất phí (fee) cho middleman.

Bitcoin và các đồng tiền điện tử khác (cryptocurrencies) sẽ định nghĩa giáo thức lớp 5 của Internet, để các máy tính có thể vận chuyển giá trị nhanh và hiệu quả như dữ liệu. Bitcoin là một công cụ vận chuyển giá trị online hữu ích, nhưng đằng sau nó chính là công nghệ Blockchain - lần đầu tiên trong lịch sử có thể thực hiện việc nhất quán một cách không tập trung (decentralized consensus).

Blockchain là một cơ sở dữ liệu nhân bản lớn của tất cả các giao dịch trong mạng lưới Bitcoin. Nó sử dụng một cơ chế nhất quán được gọi là `proof-of-network` - chống lại việc chi tiêu gấp đôi (double-spending) trong mạng lưới này. Chi tiêu gấp đôi có nghĩa là một người xấu thực hiện chi tiêu 2 lần cùng một số tiền và phủ nhận giao dịch đầu tiên của hắn.

`Proof-of-network` giải quyết vấn đề này bằng việc có các `miner` - công nhân đào mỏ trong mạng lứoi đó để xử lý các bằng chứng mật mã sử dụng phần cứng của họ. `Miners` là các node Bitcoin thực hiện xác thực các giao dịch và kiểm tra nó thông qua lịch sử blockchain. Bản ghi nhãn thời gian (timestamped) của các giao dịch luôn luôn được tạo trong mạng lưới đó. Về lý thuyết, một ai đó có thể chỉnh sửa được lịch sử blockchain của họ, nhưng với `proof-of-network`, họ cũng cần một sức mạnh tính toán lớn để xác nhận nó.  Bởi ở thời điểm hiện tại vì mạng lưới Bitcoin có sức mạnh tính toán hơn toàn bộ siêu máy tính trên thế giới gộp lại do vậy kẻ tấn công rất khó để có thể phá vỡ được mạng lưới này.

`Proof-of-network` thì quá đắt đỏ cho khối lượng tính toán , điện năng tiêu thụ nhưng nó được biết là cơ chế duy nhất để chống lại tấn công `Sybil`.
Vì vậy chúng ta có một công cụ mới được gọi là `Blockchain`, cơ sở dữ liệu nhân bản lớn của các giao dịch có thể chống lại tấn công `Sybil`. 
Đầu tiên, Blockchain giúp ta đạt được sự nhất quán phân tán mà không phải sử dụng một máy chủ trung tâm. Ta tự hỏi vậy use case của nó là gì và nó có phải là điều đúng đắn. Điều quan trọng bây giờ là hiểu cấu trúc dữ liệu của nó là một trong những điều sẽ giúp ta tạo ra một ứng dụng phân tán (dapps) có ích.

##### Decentralized Application là gì?

Hầu hết mọi người đều quen với thuật ngữ "application" là một phàn mềm. Một ứng dụng phần mềm là phần mềm định nghĩa một mục đích chỉ định. Có hàng triệu ứng dụng đang được sử dụng theo mô hình client-server. Centralized System trực tiếp điều khiển hoạt động của từng đơn vị và luồng của thông tin trong một center. 

Tất cả các cá nhân độc lập sử dụng sức mạnh trung tấm để gửi và nhận thông tin và bị ra lệnh.  Facebook, Amazon, Google hay mọi dịch vụ khác chúng ta sử dụng trên internet đều sử dụng mô hình này. Các dịch vụ lớn ta có thể coi là "The Stack". Các Stack rất hữu ích vì chúng cung cấp một dịch vụ có giá trị cho mọi người. 

*Vậy sự khác nhau giữa distributed và decentralized là gì?*

Distributed có nghĩa là việc tính toán được trải ra các node thay vì chỉ một node. Decentralized có nghĩa là không có node nào làm chỉ dẫn cho những node khác phải làm gì. Điều này có nghĩa là một hệ thống có thể là cả Centralized và Distributed.


*Một hệ thống có thể là cả distributed và centralized hay không?*

Có thể. Bitcoin là distributed vì dấu thời gian của nó là blockchain public nằm trên các máy tính khác nhau. Nó cũng là decentralized vì nếu 1 node fail thì mạng lưới vẫn hoạt động. Điều này có nghĩa là bất kể app nào sử dụng blockchain cùng với các công cụ peer-to-peer khác có thể là distributed và decentralized. 

Phần sau của chương mình không hiểu ý tưởng của tác giả lắm, mình đọc mà không thể truyền đạt lại được. Do vậy mình sẽ dừng bài này tại đây. Khi đi sâu hơn mình sẽ quay trở lại viết tiếp phần đó. 

Chương 2 sẽ nói về vận động trong hệ sinh thái của Dapps. Cố gắng tuần sau sẽ viết tiếp. 