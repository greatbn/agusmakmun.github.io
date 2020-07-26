---
layout: post
title: Chia sẻ scripts cài đặt Openstack phiên bản Ocata
image: "/content/images/2017/02/Screenshot-from-2017-02-26-12-46-16.png"
date: '2017-02-26 05:46:44'
tags:
- openstack
- cloud
- openvswitch
- ocata
---

Chào toàn thể 500 anh em, 22/2 vừa qua Openstack Foundation đã release phiên bản thứ 15 của Openstack. Phiên bản này được release sớm hơn 2 tháng tức là chỉ 4 tháng sau phiên bản Newton. 
Theo như Openstack Foundation, phiên bản này tập trung vào hiệu năng và độ ổn định nên sẽ không có nhiều tính năng mới nổi bật. 
Trong khi chờ documents của Openstack cập nhật, mình đã cài đặt manual và có viết scripts để tự động hóa việc này.

*Script dựa trên scripts của Vietstack, gửi lời cảm ơn tới ban quan trị của Vietstack (Vietname Openstack Group)*

**Lưu ý:**

- Script sử dụng cho Ubuntu 16.04

- Sử dụng Openvswitch thay vì LinuxBridge 

Tải script cài đặt 

```sh
git clone https://github.com/greatbn/openstack-scripts
cd openstack-scripts/ocata-scripts/scripts
```
Mô hình cài đặt đơn giản chỉ có 2 node: Controller và Compute. 

<img src=http://i.imgur.com/4HeUt8O.png>

Các thiết lập mạng như hình bên trên. 

Cấu hình mạng trong file `config.cfg`

- Node Controller 
```
#  Assigning IP for controller node
CTL_MGNT_IP=192.168.100.212
CTL_EXT_IP=192.168.1.212
CTL_MGNT_IF=ens33
CTL_EXT_IF=ens38
```

- Node Compute

```
# Assigning IP for Compute1 host
COM1_MGNT_IP=192.168.100.213
COM1_EXT_IP=192.168.1.213
COM1_MGNT_IF=ens33
COM1_EXT_IF=ens38
```

- Cấu hình gateway

```
# Gateway for EXT network
GATEWAY_IP_EXT=192.168.1.1
NETMASK_ADD_EXT=255.255.255.0
```

- Cấu hình dải mạng external

```
#### Floating IP pool
START_IP_ADDRESS=192.168.1.230
END_IP_ADDRESS=192.168.1.240
DNS_RESOLVER=8.8.8.8
PROVIDER_NETWORK_CIDR=192.168.1.0/24
PROVIDER_NETWORK_GATEWAY="$GATEWAY_IP_EXT"
```

- Cấu hình password mặc định 

```
## Password variable
# Set password
DEFAULT_PASS='Sapham123'
```

- Sau khi cấu hình các thiết lập cơ bản chạy các script trên controller theo thứ tự

##### Thực hiện trên node controller 

- Cấu hình địa chỉ IP, hostname

```sh
# bash ctl-0-ipadd.sh
```

- Cài đặt các thứ như repo, mysql, RabbitMQ

```sh
# bash ctl-1-prepare.sh
```

- Cài đặt keystone

```sh
# bash ctl-2.keystone.sh
```

- Cài đặt Glance

```sh
# bash ctl-3-glance.sh
```

- Cài đặt Nova

```sh
# bash ctl-4-nova.sh
```

- Cài đặt Neutron 

```sh
# bash ctl-5-neutron.sh
```

- Cài đặt Horizon

```sh
# bash ctl-horizon.sh
```

- Sau khi cài đặt sau chúng ta thực hiện cấu hình mạng cho interface `br-ex`, và reboot lại node controller, chuyển sang node Compute 

##### Thực hiện trên node Compute


- Cấu hình địa chỉ IP, hostname

```sh
# bash com1-ipdd.sh
```

- Cài đặt nova-compute, neutron-openvswitch-agent

```sh
# bash com1-install.sh
```

Sau khi cài đặt xong trên node compute, trở lại node Controller thực hiện lệnh sau để cập nhật node compute mới vào Cells

```sh
nova-manage cell_v2 discover_hosts
```

Một số hình ảnh sau khi cài đặt hoàn tất

- Logo mới

<img src="http://i.imgur.com/H6jJkxa.png">

- Keypair và API Access đã chuyển ra ngoài 

<img src="http://i.imgur.com/SCsHuQJ.png">


