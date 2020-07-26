---
layout: post
title: ELK STACK 101 - Beats Collect, Parse, Ship
image: "/content/images/2016/10/elkstack.png"
date: '2016-10-03 15:55:39'
tags:
- elasticsearch
- logstash
- logging
- beats
- filebeats
---

Beats là những data shipper mã nguồn mở mà ta sẽ cài đặt như các agent trên các server của ta để gửi các kiểu dữ liệu khác nhau tới Elasticsearch. Beats có thể gửi dữ liệu trực tiếp tới Elasticsearch hay tới Logstash từ đó ta có thể enrich hay archive dữ liệu.

Beats là một platform trong đó có các project nhỏ sinh ra thực hiện trên từng loại dữ liệu nhất định.

- Packetbeat: a network packet analyzer
- Topbeat: a server monitoring agent
- Filebeat: ship log files from servers
- Winlogbeat: ship windows event logs 
- Metricbeat: lighweight shipper collect metrics from OS, Services such as Apache, HAProxy, MongoDB, Nginx, ...

Mô hình Beats  Platform 

<img src="http://i.imgur.com/jYgT9XC.png">

Ở bài này tôi sẽ chỉ cấu hình filebeat dựa trên mục đích của tôi muốn thu thập log từ các file log trên các server khác nhau. 

#####FileBeat làm việc thế nào 

Khi khởi động filebeat, nó sẽ khởi chạy một hay nhiều prospector, sẽ tìm kiếm các đường dẫn của tập tin tin mà ta đã khai báo. Với mỗi môt tập tin log mà prospector tìm thấy được, Filebeat sẽ khởi chạy một harvester. Mỗi một harvester đọc một tập tin log, và gửi các bản tin log này khi có dữ liệu mới tới spooler. Spooler là nơi tổng hợp các sự kiện và gửi dữ liệu đã tổng hợp được tới output mà ta đã cấu hình trên Filebeat.

*Cấu trúc bên trong Filebeat*

<img src="http://i.imgur.com/XKCBATR.png">

#####Cài đặt và cấu hình

Tôi sẽ thực hiện trên Ubuntu Server 14.04

Tải và thêm GPG key 

```sh
curl https://packages.elasticsearch.org/GPG-KEY-elasticsearch | sudo apt-key add -
```
Thêm repository chưa filebeat 

```sh
echo "deb https://packages.elastic.co/beats/apt stable main" |
a /etc/apt/sources.list.d/beats.list
```

Cập nhật Repository và cài đặt filebeat 

```sh
sudo apt-get update && sudo apt-get install -y filebeat
```
Cấu hình filebeat khởi chạy cùng hệ thống

```sh
sudo update-rc.d filebeat defaults
```

Thực hiện câu hình cho filebeat đọc các file log. Dùng trình soạn thảo để mở tập
tin cấu hình của filebeat ở đường dẫn “/etc/filebeat/filebeat.yml”

Định nghĩa các đường dẫn tới tập tin log.

Một cấu hình cơ bản của Filebeat là sử dụng một prospector cho một đường dẫn.

<img src="http://i.imgur.com/xdR0l1t.png">

Prospectors trong ví dụ này sẽ tìm kiếm tât cả các tập tin trong đường dẫn
“/var/log” và các tập tin có phần mở rộng là “.log”
- Cấu hình output cho filebeat.

Nếu ta muốn sử dụng Logstash làm nơi phân tách các thông điệp thì ở mục output trong tập tin cấu hình Filebeat ta sẽ khai báo địa chỉ IP và port của Logstash. Lưu ý port này sẽ trùng với port mà ta sẽ mở trên logstash.

<img src="http://i.imgur.com/YHyn5W1.png">

Nếu ta muốn filebeat gửi tới elascticsearch luôn thì ta có thể cấu hình trong mục elasticsearch như sau:

<img src="http://i.imgur.com/k8EzDN6.png">

Ở bài này tôi sẽ sử dụng logstash để thực hiện bóc tách thông điệp. Do vậy tôi sẽ cấu hình gửi tới logstash. Lúc này trên Logstash ta cần cấu hình input để filebeat có thể kết nối vào và gửi dữ liệu tới.

Trên máy chủ cài đặt logstash, mở tập tin cấu hình của logstash trong thư mục “/etc/logstash/conf.d”. Thêm vào phần input như sau:

<img src="http://i.imgur.com/C1kxWBA.png">

Khởi động lại logstash trên máy chủ cài đặt logstash.

```sh
service logstash restart
```

Khởi động lại filebeat trên máy remote

```sh
service filebeat restart
```
Truy cập vào kibana lúc này ta sẽ thấy dữ liệu từ filebeat trả về như sau:


<img src="http://i.imgur.com/AHY4qCy.png">
