---
layout: post
title: MacVlan và IPVlan network plugin trên Docker
image: "/content/images/2016/08/docker_networking-banner1.png"
date: '2016-08-13 14:59:57'
tags:
- docker
- macvlan
- ipvlan
- networking
- network
---

####MACVlan và IPVlan

Macvlan và IPvlan là những `Linux Network Driver` cho phép interface của host trực tiếp expose tới các VM hay các container chạy trên host đó. 

#####Linux Bridge
Để hiểu hơn ta xét trường hợp sử dụng Linux Bridge (LB)trong môi trường ảo hóa network. LB hoạt động giống như một switch vậy lý bình thường. Khi triển khai LB, các VM hay các Container sẽ kết nối tới `bridge` và `bridge` sẽ kết nối với bên ngoài. Hình dưới tôi mô tả 2 VM kết nối tới bridge với eth0 cung cấp kết nối ra bên ngoài

<img src="http://i.imgur.com/Mbh9uv5.png">
#####Macvlan

Macvlan cho phép một card interface vật lý có nhiều địa chỉ Mac và địa chỉ IP sử dụng `macvlan sub-interface`. Đó là sự khác nhau khi tạo sub-interface trên 1 interface vật lý sử dụng vlan. Với vlan sub-interface mỗi sub-interface thuộc về một vùng L2 khác nhau sử dụng vlan và tất cả sub-interface có cùng địa chỉ MAC. Với macvlan, mỗi sub-interface sẽ có địa chỉ MAC và IP khác nhau và có thể expose trực tiếp tới mạng bên ngoài. Interface của macvlan được sử dụng cho các ứng dụng ảo hóa, và mỗi mỗi interface vlan kết nối tới 1 container hay một VM. Mỗi container hay VM có thể lấy IP thông qua một DHCP Server bên ngoài. 

MacVlan có 4 kiểu (Bridge, VEPA, Private, Passthrough). `Bridge` là kiểu hay được sử dụng nhất. 
Hình dưới cả 2 VM sẽ trực tiếp expose với `eth0` sử dụng macvlan sub-interface

<img src="http://i.imgur.com/WzpzXFs.png">

#####IPVlan

IPVlan tuơng tự với MacVlan nhưng khác nhau ở điểm các endpoint có cùng địa chỉ MAC. IPVlan hỗ trợ L2 và L3 mode. 

Với IPVlan L2 Mode, Mỗi endpoint sử dụng chung địa chỉ MAC nhưng khác địa chỉ IP.

Với IPVlan L3 Mode, Các packet được route giữa các endpoint. Vì vậy ở mode này cho ta sự mở rộng tốt hơn.

####Docker với macvlan và ipvlan network plugin

Docker đã hỗ trợ macvlan và ipvlan driver ở phiên bản 1.11(experimental). Và ở phiên bản 1.12 macvlan plugin này đã được đưa vào chính thức. Còn ipvlan vẫn ở mức experimental.

Do đó Ở bài này ta sẽ sử dụng macvlan network plugin cho các container để chúng có thể giao tiếp với nhau giữa các host khác nhau.

Về IPVlan khi docker release phiên bản hỗ trợ chính thức tôi sẽ viết. 
(Có thể sử dụng phiên bản thử nghiệm: `curl -fsSL https://experimental.docker.com/ | sh`)

Hình sau minh họa mô hình macvlan và ipvlan tôi sẽ sử dụng trong bài này

<img src="http://i.imgur.com/zUolPzQ.png">

Ta sẽ có các thiết lập sau:

-  Sử dụng docker-machine tạo ra 2 node

- Allow All cho promiscuous (Virtualbox) trên card 2 của 2 node. Điều này sẽ cho phép các container trên 2 node giao tiếp được với nhau thông qua card này. 

- Thực hiện tạo 4 container trên mỗi node. 2 container trên vlan100 và 2 container còn lại trên vlan 200

- Docker sử dụng là version 1.12

######Tạo 2 node sử dụng docker-machine 

Thực hiện trên máy remote

```
docker-machine create -d virtualbox node-1
docker-machine create -d virtualbox node-2
```

Mở Virtualbox allow all promiscuous mode như hình

<img src="http://i.imgur.com/F0x3HBl.png">

######Với macvlan

- Tạo network `macvlan100` và `macvlan200` trên `node-1`

```
docker@node-1:~$ docker network create -d macvlan \
> --subnet=192.168.0.0/16 \
> --ip-range=192.168.11.0/24 \
> -o macvlan_mode=bridge \
> -o parent=eth1.100 \
> macvlan100

docker@node-1:~$ docker network create -d macvlan \
> --subnet=192.168.0.0/16 \
> --ip-range=192.168.11.0/24 \
> -o macvlan_mode=bridge \
> -o parent=eth1.200 \
> macvlan200

docker@node-1:~$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
fc1e9a0c3b6a        bridge              bridge              local               
34eb4d1c2df4        host                host                local               
7839f99706b9        macvlan100          macvlan             local               
344fe9202e85        macvlan200          macvlan             local               
1d0c342786f0        none                null                local       

```

- Tạo network `macvlan100` trên `node-2`

```
docker@node-2:~$ docker network create -d macvlan \
> --subnet=192.168.0.0/16 \
> --ip-range=192.168.10.0/24 \
> -o macvlan_mode=bridge \
> -o parent=eth1.100 \
> macvlan100

docker@node-2:~$ docker network create -d macvlan \
> --subnet=192.168.0.0/16 \
> --ip-range=192.168.10.0/24 \
> -o macvlan_mode=bridge \
> -o parent=eth1.200 \
> macvlan200

docker@node-2:~$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
d16fd81985b0        bridge              bridge              local               
4fc16f6fa24f        host                host                local               
40012d92530e        macvlan100          macvlan             local               
be8991969c6c        macvlan200          macvlan             local               
b73a822f2db8        none                null                local  
```

- Thực hiện khởi tạo 2 container dùng mạng vlan100 trên node-1 và kiểm tra kết nối 

```
docker run -it --net=macvlan100 --rm --name c1-100 alpine /bin/sh
docker run -it --net=macvlan100 --rm --name=c2-100 alpine /bin/sh
```
Container `c1-100`

<img src="http://i.imgur.com/64GwxUQ.png)">

Container `c2-100`

<img src="http://i.imgur.com/Q7Rq8EG.png)">


Như vậy 2 container ở node 1 cùng vlan100 đã thông nhau.

- Thực hiện khởi tạo 2 container dùng mạng vlan100 và kiểm tra kết nối

```
docker run -it --net=macvlan100 --rm --name c3-100 alpine /bin/sh
docker run -it --net=macvlan100 --rm --name=c4-100 alpine /bin/sh
```

Container `c3-100`

<img src="http://i.imgur.com/w76NvBy.png">

Container `c4-100`

<img src="http://i.imgur.com/pnIb7CP.png">

Vậy ta đã có 4 container có thể kết nối với nhau thông qua 2 node riêng biệt

- Kiểm tra kết nối giữa `vlan100` và `vlan200`

<img src="http://i.imgur.com/2NZA2Sy.png">

Các container ở khác vlan sẽ không thể kết nối được với nhau. 

######Tổng kết

Với việc hỗ trợ macvlan driver docker cung cấp cho operator khả năng tích hợp network của Docker theo một cách đơn giản và dễ dàng vào hạ tầng mạng đang có.

Tài liệu tham khảo

- https://github.com/docker/libnetwork/blob/master/docs/macvlan.md

- https://github.com/docker/docker/blob/master/experimental/vlan-networks.md