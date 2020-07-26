---
layout: post
title: SSL/TLS cho mọi nhà
image: "/content/images/2016/09/HTTPS_icon.png"
date: '2016-09-18 14:22:02'
tags:
- ssl-tls
- https
- google-chrome
- letsencrypt
---

Đã lâu rồi cũng được 2 tuần từ lần cuối viết bài thì giờ mình mới quay lại viết bài tiếp theo. Đợt này do mình bận cũng như không có học được nhiều cái mới. Do vậy cũng không có gì để chia sẻ. Sẵn có thông tin đầu năm 2017 Google Chrome sẽ cảnh báo người dùng truy cập vào các website không sử dụng https.

<img src=http://i.imgur.com/0TPl7rm.png>

Do vậy mình sẽ viết một bài về cách nhanh nhất để có SSL/TLS free sử dụng Let's Encrypt. 

Mình thực hiện trên CentOS 7 và HTTPD. Các OS và Webserver khác có thể sẽ tuơng tự.

Thực hiện tải `certbot` từ trang chủ và set quyền execute cho file `certbot-auto`

```sh
wget https://dl.eff.org/certbot-auto
chmod a+x certbot-auto
```

Thực hiện generate Certificate và Private Key. Certbot Có thể thực hiện công việc cấu hình cho apache tự động. Nhưng theo mình là sau khi tạo Cert ta sẽ cấu hình bằng tay. Ta thực hiện lệnh sau

```sh
./certbot-auto certonly -standalone -d https.sapham.net
```

Thay tên miền của bạn vào sau `-d`

Sau khi chạy lệnh trên ta có thông báo như sau:

<img src="http://i.imgur.com/v8w7ls9.png">

Hạn của Certificate này là tới 17-12-2016. Tức là một Cert sẽ expire sau 3 tháng.

Cert và các key sẽ nằm trong thư mục `/etc/letsencrypt/live/<your_domain>`

<img src="http://i.imgur.com/J9ktzOH.png">


- cert.pem: Certificate cho domain của bạn
- chain.pem: Let's Encrypt certificate
- fullchain.pem: Nội dung bao gồm `cert.pem` và `chain.pem`
- privkey.pem: Private key của Certificate

Trên Vhost của apache ta cần cấu hình https ta sẽ khai báo các đường dẫn như sau 


<img src="http://i.imgur.com/0zDHVhc.png">

Sau đó restart lại apache và truy cập vào domain của bạn. Nhớ thêm https vào nhé 

<img src="http://i.imgur.com/vjZgaXt.png">

P/s: Ta có thể thực hiện việc auto renew bằng lệnh `./certbot-auto renew` kết hợp crontab. 

Ví dụ: 
`./certbot-auto renew --standalone --pre-hook "service httpd stop" --post-hook "service httpd start"`