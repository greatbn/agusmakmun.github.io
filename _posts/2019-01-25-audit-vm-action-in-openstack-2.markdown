---
layout: post
title: Audit VM action trong Openstack
date: '2019-01-25 04:41:43'
---


Đối với các instance (VM) trên Openstack, ta có thể có nhiều hành động để thực hiện với các instance đó. 

Ví dụ các action như:

- Create 
- Reboot 
- Hard Reboot 
- Resize 
- Shutdown
- ...

Với mỗi thao tác này thì openstack đều ghi lại lịch sử thời gian của hành động đó


<img src="https://i.imgur.com/zHGOQrg.png">

Ở hình trên ta thấy 2 action `create` và `reboot` được thực hiện với instance `49dbfbd2-c3cc-4a1b-825d-e63d4acaf90c`


Ta sẽ kiểm tra xem action reboot kia do user nào thực hiện bằng lệnh `nova instance-action <instance_id> <request_id>`


<img src="https://i.imgur.com/ErBFVH0.png">

Thực hiện tiếp action shutdown instance

<img src="https://i.imgur.com/tc8b9uG.png">


-> User thực hiện reboot và shutdown có id `a77316d14b9b4d939423058371cc8632`



Vậy đối với các action thực hiện bên trong instance thì sao, ví dụ tắt instance đi 

Ta sẽ thực hiện shutdown instance từ bên trong và kiểm tra xem openstack có track được các action này không nhé 

Vì lúc này khi tắt bên trong, trạng thái của instance chuyển sang shutoff nên chắc chắn openstack cũng sẽ track để cập nhật lại trạng thái của instance.

Kiểm tra action và user thực hiện 

Đơn giản hơn ta có thể xem trên dashboard, phần chi tiết 1 instance 


<img src="https://i.imgur.com/KEoHknE.png">


Tại thời điểm `Jan. 25, 2019, 4:38 a.m.	` bị stop và không có `User ID` thực hiện, do instance bị stop từ bên trong.

