---
layout: post
title: Login Grafana thông qua Google OAuth
image: "/content/images/2016/12/68747470733a2f2f662e636c6f75642e6769746875622e636f6d2f6173736574732f31303939392f323531383832302f64626231313031612d623436382d313165332d393162662d3234326339633633326330372e504e47.png"
date: '2016-12-21 18:31:47'
tags:
- monitoring
- grafana
- google-oauth
---

Xin chào các bạn, sau hơn 3 nghìn năm bận rộn trong công việc và học tập. Có lẽ mình sẽ quay lại viết bài tiếp trên blog của mình. Mình nhắc lại rằng, những bài viết của mình chỉ là để notes lại những điều mình cần notes và mình muốn chia sẻ cho tất cả mọi người.

Thôi không chém gió nữa, như tiêu đề của bài viết thì mình thấy khá nhiều đơn vị sử dụng Grafana như một công cụ Visualize khá hiệu quả. Ở phiên bản 4 thì Grafana còn hỗ trợ các alert. Do vậy việc quản lý tài khoản cho các sysadmin trở nên phức tạp hơn, và không phải tổ chức nào cũng có AD hay LDAP để tích hợp vào cả. Grafana hỗ trợ việc tích hợp login bằng Google OAuth hoặc Github. Nhưng ở bài này mình sẽ chỉ hướng dẫn đối với Google OAuth, đối với Github cũng tương tự.

Đầu tiên ta cần phải tạo một ứng dụng trên [Google Developer Console](https://console.developers.google.com). 
Chọn Mục `Credentials`  và chọn `Create a project` như hình 
<img src="http://i.imgur.com/VLyLE2u.png">

Nhập tên cho project và nhấn Create 

<img src="http://i.imgur.com/0DvWjqk.png">

Tiếp tục chọn `Create credentials` và chọn `OAuth client ID` như hình dưới 

<img src="http://i.imgur.com/Y9uScw3.png">

Ở bước này nếu ta chưa cấu hình tên cho sản phẩm thì sẽ được thông báo như sau nhấn chọn `Congiure consent screen` để cấu hình 

<img src="http://i.imgur.com/D1xVXzK.png)">

Điền tên của project và nhấn Save để lưu 

<img src="http://i.imgur.com/AFXawqy.png">

Chọn `Web Application` và nhập thông tin như dưới hình và nhấn Create

<img src="http://i.imgur.com/Ad3yJa3.png">

Sau khi tạo thành công Google sẽ trả về cho ta Client ID và Secret Key. Bạn phải giữ bí mật thông tin này. Và ta sẽ thực hiện cấu hình trên Grafana Server.

<img src="http://i.imgur.com/mw2OqyN.png)">

Tiếp theo ta cần xác minh domain đang trỏ tới grafana server. Chọn tab `Domain Verification`, chọn `Add domain` và nhập Domain như hình

<img src="http://i.imgur.com/Ojud8T4.png)">

Bước này ta sẽ cần tải lên tập tin xác thực. Bước này mình sẽ không viết vào đây.

Tiếp theo mở file cấu hình grafana thông thường tại đường dẫn `/etc/grafana/grafana.ini`

Tại Section `server` ta cần cấu hình `domain` và `root_url` trỏ về domain đang dùng

Tại section `auth.google` cấu hình như sau, sử dụng `client_id` và `client_secret` ở bước trước 

<img src="http://i.imgur.com/LperJEr.png)">

- Lưu ý: 
  - Phải cấu hình `allow_sign_up = true`
  - `allowed_domains`: Cho phép domain nào được đăng nhập, cách nhau khoảng trắng. 

Restart grafana và kiểm tra

```
service grafana-server restart
```

<img src="http://i.imgur.com/cebVCYP.png)">
 