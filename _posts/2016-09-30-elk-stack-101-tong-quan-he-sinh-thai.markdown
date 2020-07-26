---
layout: post
title: ELK Stack 101  - Tổng quan hệ sinh thái
image: "/content/images/2016/09/elkstack.png"
date: '2016-09-30 16:27:26'
tags:
- elasticsearch
- logstash
- kibana
- logging
- monitoring
---

####1. Logstash
Logstash là một công cụ mã nguồn mở thu thập dữ liệu có khả năng liên hợp theo thời gian thực. Logstash có thể hợp nhất dữ liệu từ các nguồn khác nhau và chuẩn hóa dữ liệu ở phần xử lý tiếp theo. Loại bỏ và đồng hóa tất cả dữ liệu đó trong một số use case cần phân tích và thể hiện trên biểu đồ.

- **INPUT**: Nó có thể lấy đầu vào từ TCP/UDP, các file, từ syslog, Microsoft Windows EventLogs, STDIN và từ nhiều nguồn khác. Chúng ta có thể lấy log từ các ứng dụng trên môi trường của chúng ta rồi đẩy chúng tới Logstash.

- **FILTER**: Khi những log này tới Server Logstash, có một số lượng lớn các bộ lọc mà cho phép ta có thể chỉnh sửa và chuyển đổi những event này. Ta có thể lấy ra các thông tin mà ta cần từ những event log. 
- **OUTPUT**: Khi xuất dữ liệu ra, Logstash hỗ trợ rất nhiều các đích tới bao gồm TCP/UDP, email, các file, HTTP, Nagios và số lượng lớn các dịch vụ mạng. Ta có thể tích hợp Logstash với các công cụ tính toán số liệu (metric), các công cụ cảnh báo, các dạng biểu đồ, các công nghệ lưu trữ hay ta có thể xây dựng một công cụ trong môi trường làm việc của chúng ta.
####2. Elasticsearch 
Elasticsearch là một công cụ nguồn mở tìm kiếm full-text và phân tích có khả năng mở rộng cao. Nó cho phép lưu trữ, tìm kiếm và phân tích khối lượng lớn của dữ liệu nhanh chống và tương đối gần vời thời gian thực. 

Nó thường được sử dụng như một engine hay công nghệ phía bên dưới tạo ra sức mạnh cho các ứng dụng bên trên có các tính năng và yêu cầu tìm kiếm phức tạp.

####3. Kibana 
Kibana là một nền tảng phn tích và hiển thị mã nguồn mở được thiết kế để làm việc với Elasticsearch. Kibana được viết bằng HTML và Javascript. Nó tận dụng khả năng tìm kiếm và index mạnh mẽ của Elasticsearch để hiển thị các giao diện mạnh mẽ cho người dùng. Từ kinh doanh thông minh tới việc kiểm tra lỗi theo thời gian thưc. Nhiệm vụ của kibana là tạo ra các biểu đồ, bản đồ địa lý, các bảng biểu,...

Kibana làm cho việc hiểu các khối lượng dữ liệu lớn trở nên dễ dàng hơn. Giao diện duyệt đơn giản cho phép ta tạo và chia sẻ một biển đồ nhanh chóng mà có thể hiển thị sự thay đổi bằng việc truy vấn vào Elasticsearch theo thời gian thực.

####4. Hệ sinh thái ELK STACK

<img src=http://i.imgur.com/USwOsjz.png>

Bên trên là hệ sinh thái của Logstash sẽ có 4 thành phần: 

- Shipper: Gửi các event tới logstash. Máy remote của ta thường sẽ chỉ chạy thành phần này. 
- Broker và Indexer: Nhận và index các event.
- Search và Storage: Cho phép tìm kiếm và lưu trữ các event.
- Web Interface: Một giao diện web để thực hiện các hiển thị và tìm kiếm từ Search và Storage. 