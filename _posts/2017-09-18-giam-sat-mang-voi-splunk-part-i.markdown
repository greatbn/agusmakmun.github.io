---
layout: post
title: Giám sát mạng với Splunk - Part I
image: "/content/images/2017/09/logo_splunk_white_high.png"
date: '2017-09-18 07:24:37'
tags:
- monitoring
- giam-sat
- splunk
- cloud
- security
---

######Splunk là gì?
Splunk là môt công cụ vô cùng mạnh mẽ để tìm kiếm, khám phá và hiển thị tất cả các kiểu của dữ liệu.

Splunk đang phổ biến nhanh và nhiều hơn tại các doanh nghiệp, cả lớn và nhỏ. Các nhà phân tích, các người quản lý, sinh viên có thể nhanh chóng học cách sử dụng dữ liệu từ hệ thống, các mạng, lưu lượng truy cập web và các dữ liệu mạng xã hội của họ để tạo các báo cáo hấp dẫn và có nhiều thông tin.

Tên "Splunk" được lấy cảm hứng bởi quá trình khám phá các hang động hay splunking giúp các nhà phân tích, vận hành, lập trình viên khám phá dữ liệu từ tổ chức bằng việc thu thập  phân tích và báo cáo dựa trên splunk. 

Splunk là một công ty đa quốc gia được sáng lập bởi Michael Baum, Rob Das và Erik Swan có một sản phẩm chính là "Splunk Enterprise"

######Tính năng của Splunk 

*Định dạng log*

Splunk có thể định dạng từng loại log khác nhau, các loại log được sinh ra từ những ứng dụng khác nhau. Mỗi loại log sẽ được đánh kiểu theo từng loại cụ thể (sourcetype). Thông thường khi log được xử lý bởi Splunk, nó thực hiện 2 công việc chính là:

- Phân tách log 

- Đánh chỉ mục

Mỗi loại log có một cách sắp xếp dữ liệu khác nhau do vậy khi các bản tin tới Splunk server nó sẽ được phân tách thành các trường, công việc này hỗ trợ rất nhiều cho quá trình tìm kiếm và lọc thông tin. Do vậy bước này cực kì quan trọng đối với hệ thống giám sát tập trung.

*Thu thập dữ liệu*

Quá trình thu thập dữ liệu với Splunk được nâng cao vì hệ thống của splunk làm cho việc thu thập dữ liệu đơn giản hơn từ nhiều kiểu dữ liệu khác nhau của các hệ thống máy tính. 
Dữ liệu có thể từ:

- Các tập tin, thư mục
- Cổng cổng TCP/UDP 
- Các Script

Để thêm dữ liệu vào Splunk trên giao diện Web ta có 3 cách như hình sau

<img src="https://i.imgur.com/ympbRvl.png">


- Upload: Ta có thể upload các tập tin log hay các tập tin log từ máy tính

- Monitor: Splunk sẽ thực hiện giám sát trên chính máy đang cài Splunk. Với tùy chọn này ta lại có các kiểu như: Tập tin, thư mục; lấy từ HTTP; các cổng TCP/UDP; các Script

<img src="https://i.imgur.com/8XhXXCI.png">


- Forward: Các nguồn dữ liệu sẽ được lấy từ các Splunk Forwarder, ở phần này ta cần phải cấu hình trên server cần lấy dữ liệu và triển khai splunk forwarder như một client. Khi đó Splunk có thể điều khiển được splunk forwarder


*Đánh chỉ mục dữ liệu*

Splunk có thể đánh chỉ mục cho rất nhiều kiểu dữ liệu. Các nguồn dữ liệu thông thường:

- Các dữ liệu có cấu trúc: Các tập tin CSV, JSON hay XML 
- Các dịch vụ Web: Apache, IIS
- Các phần mềm vận hành IT: Nagios, NetApp, Cisco USC
- Dịch vụ cơ sở dữ liệu: Oracle, MySQL, Microsoft SQL Server
- Mạng và An toàn: Syslog, SNMP, các thiết bị của Cisco, Snort
- Các nền tảng ảo hóa: VMWare, Xen Desktop, XenApp, Hyper-V
- Các dịch vụ ứng dụng: JMX & JMS, WebLogic, WebSphere,
- Nền tảng của Microsoft: Exchange, Active Directory, Sharepoint

*Tìm kiếm thông tin*

Cốt lõi của nền tảng Slunk là SPL - Splunk's Search Processing Language. SPL là một ngôn ngữ có khả năng vô hạn và học nó cũng không quá phức tạp. Nó đưa cho ta sức mạnh để hỏi bất kì câu hỏi nào với bất kỳ dữ liệu nào của máy. 

SPL hòa hợp những khả năng tốt nhất của SQL và cú pháp xử lý liên hợp trên Unix cho phép ta:

- Truy cập tất cả dữ liệu trong định dạng gốc của nó.
- Tối ưu hóa cho các sự kiện chuỗi thời gian
- Sử dụng cùng một ngôn ngữ cho mô phỏng

SPL cung cấp hơn 140 lệnh cho phép tìm kiếm, tương quan, phân tích và mô phỏng bất kì dữ liệu nào. Một ngôn ngữ mạnh mẽ có thể tóm tắt trong 5 ý chính sau:

- Tìm những thứ nhỏ nhất trong Haystack
Tìm kiếm cho các từ khóa và lọc thông qua tập các dữ liệu 

- Mô phỏng dữ liệu địa lý theo thời gian thực 

Sử dụng lệnh "iplocation" để gián địa chỉ IP theo kinh độ và vĩ độ, "geostats" để gán các số liệu theo thời gian thực.

- Dự đoán, biểu đồ và mô phỏng số liệu

Sử dụng lệnh "stats" mạnh mẽ với hơn 20 lựa chọn tính toán số liệu khác nhau. Sau đó các biểu đồ, mô phỏng thể hiện kết quả các liệu thông qua khoảng thời gian 

- Máy học và phát hiện dị thường

Sử dụng phát hiện dị thường để mở ra các hành động và các sự kiện bất thường. Xây dựng và áp dụng mô hình máy học với các lệnh "fit" và "apply"

<img src="https://i.imgur.com/BML5s2C.png)">

* Giám sát mạng và cảnh báo *

Splunk cung cấp cho người dùng một cơ chế cảnh báo dựa trên việc tìm kiếm các thông tin do chính người sử dụng đặt ra. Khi có vấn đề liên quan tới hệ thống phù hợp với các tiêu chí đó thì hệ thống sẽ cảnh báo tới người dùng.

Hiện tại Splunk ta có thể đặt cảnh báo Splunk qua:

- Một Log Event
- Chạy một script 
- Gửi Email
- Gửi một HTTP POST 
- Thậm chí có thể gọi điện hoặc gửi tin nhắn qua số điện thoại của người dùng nếu ta cấu hình

<img src="https://i.imgur.com/nqlUuZt.png">

Đặc biệt ta có thể sử dụng hỗn hợp các lựa chọn này cùng một lúc. Ví dụ ta có thể vừa gửi một HTTP POST  vừa gửi một email.

<img src="https://i.imgur.com/1hKH88a.png">

*Khắc phục sự cố*

Splunk còn cung cấp một cơ chế tự động khắc phục sự cố với các vấn đề xảy ra bằng việc cấu hình để tự đọng chạy các tập tin Script mà người dùng đã tạo.

Ví dụ ta có một script chặn các địa chỉ IP đang thực hiện tấn công DoS vào hệ thống của ta. 

Ta sẽ thực hiện đếm các request đến từ cùng một địa chỉ IP, nếu số lượng request vượt quá số lượng request cho phép trong 1s thì script sẽ tự chạy, và ta có thể bảo vệ hệ thống của ta khỏi cuộc tấn công đó. Công việc hoàn toàn tự động

<img src="https://i.imgur.com/FyHNquG.png">

*Hiển thị thông tin*

Splunk cung cấp một cơ chế hiển thị trực quan giúp người sử dụng có thể dễ dàng hình dung về tình trạng của hệ thống, đưa ra các đánh giá về hệ thống. 

Người sử dụng có thể hiển thị các biểu đồ dựa trên kết quả của một câu lệnh tìm kiếm. Từ đó có một cách nhìn trực quan, dễ hiểu, rõ ràng nhất đối với hệ thống

<img src="https://i.imgur.com/Nb4yJY2.png">


Như hình trên ta có thể thấy rằng Trung Quốc thực hiện rất nhiều các cuộc tấn công SSH vào hệ thống của chúng ta. 

Sau khi tìm kiếm và hiển thị thông tin, ta có thể lưu lại vào Dashboard, ta chọn "Save As" tiếp theo chọn "Dashboard Panel"

<img src="https://i.imgur.com/FY8mer7.png">

Ở đây ta có thể lựa chọn tạo mới Dashboard hay Thêm vào Dashboard đã tồn tại. Các công việc rất đơn giản và dễ dàng 

Splunk còn tự động kết xuất ra các báo cáo với nhiều loại định dạng một cách chuyên nghiệp. Để xuất một cáo cáo ta cần vào phần Dashboard mà ta muốn xuất báo cáo chọn "Export"

*Phát triển*

Splunk cung cấp các API hỗ trợ người dùng phát triển ứng dụng của họ trên Splunk. Một số API điển hình như 

- Splunk SDK: Cung cấp các SDK trên nền tảng Python, Java, JS, PHP, Ruby, C#

Splunk SDK được viết dựa trên Splunk REST API. Những thứ mà ta có thể làm đối với Splunk SDK là: tích hợp với các công cụ báo cáo và cổng thông tin của bên thứ 3, Đẩy log trực tiếp tới Splunk, Tích hơp kết quả tìm kiếm của Splunk vào ứng dụng của người dùng, Giải nén dữ liệu, Xây dựng một giao diện Web theo ý thích của người dùng,... 

- Shep: Splunk Hadoop Intergration - Đây là sự kết hợp giữa Splunk và Hadoop

Sử dụng Hadoop như một kho lưu trữ của Splunk có các tính năng sau:
```
    - Map Reduce Job trong Hadoop có thể sử dụng dữ liệu của Splunk 
    - Truyền dữ liệu từ Splunk tới HDFS trong thời gian thực
    - Ngôn ngữ tìm kiếm được tích hợp với Hadoop
    - Giám sát các tập tin và các thư mục của HDFS cho việc đánh chỉ mục của Splunk 
```

- Splunk PowerShell Resource Kit: Bộ công cụ hỗ trợ việc mở rộng và quản lý hệ thống

Sử dụng Splunk PowerShell Resource Kit, người quản trị hệ thống Windows có thể quản lý và mở rộng môi trường Splunk để hỗ trợ nhiều công việc. Với phiên bản đầu tiên của bộ công cụ này, các người quản tri có thể quản lý topo của Splunk, cấu hình các thành phần bên trong và mượn bộ máy tìm kiếm của Splunk từ một phiên của PowerShell

Các nhóm lệnh có thể thực hiện trên PowerShell để tương tác với Splunk bao gồm:

```
- Kiểm tra và quản lý các dịch vụ Splunk
- Tìm kiếm trên Splunk
- Triển khai Splunk
- Quản lý các lớp của các máy chủ trong Splunk 
```