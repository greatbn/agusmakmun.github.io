---
layout: post
title: ZeroDB - End to end encrypted database
image: http://i.imgur.com/LbgGr6v.png
date: '2017-05-04 16:22:26'
tags:
- zerodb
- database
- encryped
---

#### ZeroDB

Lợi ích của môi trường điện toán đám mây đối với doanh nghiệp bao gồm giá tiền, hiệu năng và độ mở rộng, khi họ tận dụng những lợi ích này để xây dựng cơ sở hạ tầng của họ tại một nhà cung cấp dịch vụ điện toán đám mây nào đó. Tuy nhiên, các vấn đề bảo mật dữ liệu và các dữ liệu nhạy cảm khi đưa lên điện toán đám mây làm cho các doanh nghiệp quan tâm hàng đầu. Khi các cơ chế mã hóa mạnh có thể làm giảm bớt các vấn đề này bằng việc đảm bảo dữ liệu được bí mật (confidentiality), một khi dữ liệu được mã hóa, nó có thể không hữu dụng nữa. 

ZeroDB cung cấp tính bí mật cho dữ liệu được lưu trữ trên các máy chủ cơ sở dữ liệu. Giả định rằng, các client có sở hữu dữ liệu và các khóa mã hóa liên quan. Các client này muốn dữ liệu được mã hóa trên các máy chủ không tin cậy, trong khi vẫn duy trì tính hữu ích của dữ liệu. Trong khi kẻ tấn công thì không thể đọc được dữ liệu này.

#### Cơ chế hoạt động

Quy tắc cơ bản của ZeroDB là mã hóa đầu cuối, giao thức truy ván của zerodb như sau: 

- Client tương tác với máy chủ cơ sở dữ liệu trong khi thực hiện các truy vấn thông qua một loạt vòng lặp.

- Một chỉ mục đã mã hóa được lưu trên máy chủ ở duới dạng B-Tree (xem hình sau)

<img src="http://i.imgur.com/KSoRySm.png)">

- Client đi theo chỉ mục này để nhận các bản ghi đã mã hóa cần thiết

- Các đường đi này sẽ gia tăng với mỗi vòng lặp dựa theo bước đi xuống của chỉ mục trên B-Tree

- Các chỉ mục bao gồm các bucket (lô) đã bị mã hóa trước khi chúng được upload lên máy chủ và chỉ được giải mã tại phía client

#### Một truy vấn trong ZeroDB

Trong ZeroDB, các chỉ mục được tổ chức và mã hóa dưới dạng B-Tree. Một B-Tree bao gồm các bucket đã mã hóa. Mỗi chỉ mục có root (gốc), branch( nhánh) hoặc leaf (lá). Leaf là một điểm của B-Tee, lưu trữ các đối tượng thực sự. Vì vậy, việc truy vấn hoặc tìm kiếm trong cơ sở dữ liệu này cơ bản là đi dọc theo cây.

Để có thể làm cho cơ sở dữ liệu được mã hóa ở 2 đầu cuối, mà vẫn có khả năng thực hiện các truy vấn, client phải mã hóa các bucket khi tạo hay chỉnh sửa. Máy chủ lưu các bucket không biết về khóa đã được sử dụng để mã hóa. Các đối tượng tham chiếu bởi các nút lá (leaf) của các chỉ mục B-Tree cũng được mã hóa ở phía client. Kết quả này, máy chủ cũng không biết các đối tượng được tổ chức bên trong B-Tree như thế nào hay chỉ mục nào mà chúng thuộc về cả. 

Khi client thực hiện một truy vấn, nó hỏi máy chủ để trả về các bucket của cây đó và sau đó nó đi dọc các chỉ mục sau đó tăng dần mô tả như hình dưới đây.
Hình: Dãy các yêu cầu của client để đi dọc theo một chỉ mục 

<img src="http://i.imgur.com/r5YtgTw.png)">

Client lấy về bucket và giải mã nó để tìm ra mức tiếp theo nằm trong cây mà nó sẽ lấy về. Các bucket thường xuyên truy cập có thể được cache ở phía client để cải thiện tốc độ truy vấn và không làm tốn tài nguyên mạng. 

Máy chủ chịu trách nhiệm cho việc nhân bản dữ liệu, các phiên bản, các đối tượng đang bị khóa, xác thực người dùng, hạn ngạch và đảm bảo ghi một lần - đọc nhiều lần (write-once-read-many - WORM)

Client thực hiện mã hóa, giải mã và thực thi các truy vấn logic. Vì vậy, nếu máy chủ bị nhiễm mã độc, dữ liệu liệu của người dùng không thể bị giải mã.

Mặc định, ZeroDB sử dụng hệ mật AES256 hoạt động ở chế độ GCM. Các dữ liệu của ZeroDB không bị giải mã ở phía server. 

Trong ZeroDB, tất cả nội dung của cơ sở dữ liệu và các chỉ mục (index) cũng như được lưu trữ ở dạng mã hóa. Vì thế, kẻ tấn công không thể tìm hiểu được gì ngoài kích thước của cơ sở dữ liệu (bao gồm số lượng các trường). Vì vậy ZeroDB đảm bảo an ninh ngữ nghĩa chống (semantic security) lại các kẻ tấn công 

#### Cấu trúc lưu trữ của zerodb

Ở bước cài đặt, sau khi cài đặt thành công ta thực hiện khởi tạo cơ sở dữ liệu của ZeroDB tại thư mục “data”. Cấu trúc thư mục này như sau:

<img src="http://i.imgur.com/lCv9m4X.png)">

- Thư mục “conf”: lưu các cấu hình cho ZeroDB và các tài khoản người dùng
- Thư mục “db”: là nơi lưu chứa dữ liệu của cơ sở dữ liệu
- Tập tin “conf/server.zcml”: 

<img src="http://i.imgur.com/bgoSjc3.png)">

+ “address”: Cấu hình địa chỉ IP mà ZeroDB bind vào và số cổng sử dụng
+ “authentication-protocol”:  giao thức xác thực
+ “authentication-database”: đường dẫn tới cơ sở dữ liệu xác thực
+ “authentication-realm”: Tên vùng xác thực, tương tự như khái niệm domain
+ “path”: Đường dẫn tới cơ sở dữ liệu chứa dữ liệu

Dữ liệu tại máy chủ ZeroDB ở dạng bản mã

<img src="http://i.imgur.com/2QOkE72.png)">