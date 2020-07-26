---
layout: post
title: Trải nghiệm các tính năng mới trên Docker 1.12
image: "/content/images/2016/06/download.png"
date: '2016-06-21 18:09:37'
tags:
- docker
- docker-swarm
- google-kubernetes
- docker-1-12
---

Ngày hôm qua, trong sự kiện diễn ra Dockercon 16. Docker đã giới thiệu phiên bản mới Docker version 1.12 với nhiều tính năng mới.

Ở phiên bản này, Docker đã nhấn mạnh vào 4 nguyên tắc:

- Simple Yet Powerful
- Resilient
- Secure
- Optional Features and Backward Compatibility 

Tham khảo thêm tại đây: [Docker Blog](https://blog.docker.com/2016/06/docker-1-12-built-in-orchestration/)
Hôm nay mình sẽ trải nghiệm một số tính năng mới chủ yếu là trên swarm với việc tạo swarm với câu lệnh `docker swarm init` và create, scale  service 

Để cài đặt phiên bản 1.12 này ta thực hiện lệnh sau

```
curl -fsSL https://test.docker.com | sh
```

Ở bài này mình sẽ dùng 3 node chạy Ubuntu 14.04 , các node đều được cài phiên bản 1.12.

<img src="http://i.imgur.com/ZSiM9oX.png">



Thực hiện init swarm trên node `swarm-manager`

```
docker swarm init --listen-addr 10.30.0.21:2377
```
Với  `10.30.0.21` và `2377` là địa chỉ IP và Port mà swarm lắng nghe trên node swarm-mangager


Kiểm tra các node trong swarm

```
docker node list
```

<img src="http://i.imgur.com/hkaKJrk.png">


Ở đây ta thấy node `swarm-manager` là Leader

Thực hiện join 2 node `swarm-node-01` và `swarm-node-02`

```
docker swarm join 10.30.0.21:2377
```
Trên node-01
<img src="http://i.imgur.com/tpENydC.png">

Và trên node-02

<img src="http://i.imgur.com/Whoue9J.png">

Sau đó ta kiểm tra các node trong swarm

<img src="http://i.imgur.com/d0tqj0a.png">


Trên node swarm-manager tạo một service 

Ví dụ tôi sẽ tạo một service image alpine thực hiện `ping 8.8.8.8`
```
docker service create --name ping alpine ping 8.8.8.8
```

<img src="http://i.imgur.com/iZz3PMl.png">

Ta đã tạo một service với tên `ping`. Kiểm tra container trong service này đang chạy trên node nào

```
docker service tasks ping
```

<img src="http://i.imgur.com/vhGSoQQ.png">

Có một container chạy trên node `swarm-manager`

Kiểm tra logs trên container này

<img src="http://i.imgur.com/mq7aT4F.png">

Ok. Nó đã hoạt động

Ta có thể thực hiện scale service này tức là ta sẽ tạo ra nhiều container trong service ping

```
docker service update --replicas 3
```

<img src="http://i.imgur.com/5ATcXsB.png">

Tóm lại, ta thấy docker mang khái niệm mới vể service khá tương đồng với Pod của Google Kubernetes