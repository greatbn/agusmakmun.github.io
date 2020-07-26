---
layout: post
title: ELK STACK 101 - Cài đặt  và cấu hình
image: "/content/images/2016/09/elkstack-1.png"
date: '2016-09-30 16:44:18'
tags:
- elasticsearch
- logstash
- kibana
- logging
- elk-stack
---

#####Mô hình cài đặt 
Tôi sẽ cài 3 thành phần Logstash, Elasticsearch và Kibana trên cùng một server. Các thành phần này đều có thể mở rộng theo chiều ngang, do vậy việc cấu hình mở rộng cũng không có gì khó khăn. 

Bài lab sau sẽ thực hiện trên Ubuntu Server 14.04
<img src="http://i.imgur.com/l93AWOu.png">
#####JAVA 
Vì Logstash và Elasticsearch được viết bằng ngôn ngữ Java, do vậy ta cần phải cài Java trên máy chủ cài đặt 2 thành phần này.

Ta sẽ thực hiện cài đặt Java 8 từ package. 
Thực hiện các lệnh sau trên máy chủ Ubuntu:
- Thêm Repository có chứa package Java 8 

```sh 
sudo add-apt-repository ppa:webupd8team/java  
```
- Cập nhật repository sau khi đã thêm

```sh
sudo apt-get update
```
- cài đặt Java

```sh
sudo apt-get install oracle-java8-installer oracle-java8-set-default  
```
- Kiểm tra 

<img src=http://i.imgur.com/SuspfGj.png>
#####Logstash 
- Tải và thêm GPG key sử dụng lệnh sau:
```sh
wget -qO - https://packages.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -  
```
- Thêm repository bằng lệnh sau 

```sh
echo "deb https://packages.elastic.co/logstash/2.3/debian stable main" | sudo tee -a /etc/apt/sources.list 
```
- Cập nhật repository sau khi đã thêm

```sh
sudo apt-get update
```
- Cài đặt logstash 

```sh
sudo apt-get install -y logstash  
```

- Để cho phép logstash khởi động cùng hệ thống ta thực hiện lệnh sau

```sh
update-rc.d logstash defaults  
```
- Để kiểm tra quá trình cài đặt đã thành công hay chưa ta khởi chạy logstash với input là dữ liệu nhập từ bàn phím và output là màn hình

```sh
cd /opt/logstash  
bin/logstash -e 'input { stdin { } } output { stdout {} }'
```
Khi đó logstash sẽ khởi động ta sẽ nhập một vài đoạn chuỗi và nó sẽ hiển thị ra màn hình

<img src="http://i.imgur.com/6WghBz0.png)">
#####Elasticsearch
- Thêm GPG key như phần logstash nếu chưa có 
- Thêm repository bằng lệnh sau 

```sh
echo "deb https://packages.elastic.co/elasticsearch/2.x/debian stable main" | sudo tee -a /etc/apt/sources.list.d/elasticsearch-2.x.list  
```
- Cập nhật repository sau khi đã thêm

```sh
sudo apt-get update
```
- Cài đặt elasticsearch
```sh
sudo apt-get install -y elasticsearch
```
Mở file cấu hình tại đường dẫn “/etc/elasticsearch/elasticsearch.yml”

Sửa các mục sau

- cluster.name: là tên của elasticsearch cluster. 
Dùng để định danh một cluster Elasticsearch và để các node tự tham giao vào cluster. 
- node.name: là tên của node hiện đang chạy 
- path.data: đường dẫn để lưu dữ liệu cho elasticsearch
- path.logs: đường dẫn lưu các tệp tin log của elasticsearch 
- network.host: là địa chỉ IP của máy chủ mà elasticsearch sẽ bind
- http.port: là port mà elasticsearch sẽ lắng nghe, ta sẽ sử dụng mặc định là 9200
Tham khảo tệp tin cấu hình cho elasticsearch của tôi như sau:
```sh 
cluster.name: saphi-elasticsearch  
node.name: node-1
path.data: /var/lib/elasticsearch  
path.logs: /var/log/elasticsearch  
```
Sau khi cấu hình thành công ta thực hiện khởi động elasticsearch 

```sh
service elasticseach restart  
```
- Kiểm tra trạng thái của cluster 

<img src="http://i.imgur.com/pYZ0Qoy.png">

#####Kibana
- Thêm GPG key như phần logstash nếu chưa có 
- Thêm repository bằng lệnh sau 

```sh
echo "deb http://packages.elastic.co/kibana/4.5/debian stable main" | sudo tee -a /etc/apt/sources.list 
```
- Cập nhật repository sau khi đã thêm

```sh
sudo apt-get update
```
- Cài đặt Kibana 

```sh
sudo apt-get install -y kibana  
```
- Sau khi quá trình cài đặt hoàn tất ta khởi động kibana 

```sh
service kibana start 
```
Ta cần cấu hình để kibana kết nối đến elasticsearch. Mở tập tin cấu hình kibana tại đường dẫn “/opt/kibana/config/kibana.yml”
Cấu hình các thông số sau 

- server.port: Cổng hoạt động của kibana, ta sẽ để mặc định là 5601 
- server.host: địa chỉ IP mà kibana sẽ bind. Mặc định là 0.0.0.0
- elasticsearch.url: địa chỉ url của elasticsearch. 

Khởi động lại kibana
```sh
service kibana start 
```
