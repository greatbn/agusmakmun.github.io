---
layout: post
title: Sử dụng Prometheus để giám sát máy chủ Linux/ Windows - Phần 1
image: "/content/images/2017/10/monitoring-with-prometheus-21-638.jpg"
date: '2017-10-10 04:09:00'
tags:
- prometheus
- cloud-native
- monitoring
- metric
---

###### Prometheus là gì?
Prometheus là một hệ thống giám sát và công cụ cảnh báo được opensource bởi Sound Cloud. 

Promethues được join vào tổ chức Cloud Native Computing vào năm 2016, là project thứ 2 sau Kubernetes.
 
###### Các tính năng

Prometheus có các tính năng chính như sau:

- Mô hình dữ liệu đa chiều - time series được xác định bởi tên của số liệu (metric) và các cập khóa - giá trị (key/value)
- Ngôn ngữ truy vấn linh hoạt
- Chỉ cần 1 máy chủ là có thể hoạt động được
- Hỗ trợ Push các time series thông qua một gateway trung gian
- Các đích giám sát có thể được phát hiện thông qua service discovery hoặc cấu hình tĩnh
- Hỗ trợ nhiều chế độ biểu đồ. 
###### Các thành phần

Một hệ sinh thái Prometheus bao gồm nhiều thành phần khác nhau. Các thành phần dưới đây có thể được tùy chọn khi triển khai

- Máy chủ Prometheus chính làm nhiệm vụ lấy (scrape) và lưu trữ dữ liệu time series
- Thư viện client cho các ứng dụng
- Push Gateway để hỗ trợ cho các job có thời gian thực hiện ngắn,. 
- Các exporter đặc biệt cho từng mục đích (cho HAproxy , StatsD, Graphite, ...)
- AlertManager: Quản lý các cảnh báo
- Và nhiều công cụ hỗ trợ khác ...

Toàn bộ thành phần của Prometheus được viết bằng Golang. 

###### Kiến trúc

Hình sau mô tả toàn bộ kiến trúc của Prometheus và các thành phần khác trong hệ sinh thái của nó.

<img src="https://prometheus.io/assets/architecture.svg">

Promethues thực hiện scrape các metric từ các job chỉ định, trực tiếp hoặc thông qua push gateway trung gian. Lưu trữ các mẫu thu thập được tại local và chạy các luật trên dữ liệu này để tạo ra time series mới hoặc tạo ra cảnh báo. 

Ta có thể sử dụng Grafana hoặc các chuơng trình khác để hiển thị các dữ liệu này.
