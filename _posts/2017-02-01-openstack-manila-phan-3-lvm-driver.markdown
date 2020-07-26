---
layout: post
title: 'Openstack manila phần 3: LVM Driver'
image: "/content/images/2016/06/openstack-1.jpg"
date: '2017-02-01 13:51:00'
tags:
- openstack
- manila
- lvm
- shared-file-system
- storage
---

#####Cấu hình manila-share với LVM

Trên node cài manila-share ta cần cài thêm gói lvm2 và nfs-kernel-server. Do vậy manila-share sẽ trực tiếp quản lý các volume trên host này. Để mở rộng ta sẽ phải cài manila-share trên tất cả các node LVM.

Mô hình

<img src="http://i.imgur.com/4K8lnyR.png">

- Với backend là LVM ta sẽ cần tạo ra một Volume Group và khai báo Volume Group này cho manila-share

- Cài đặt các thành phần

```
add-apt-repository cloud-archive:mitaka
apt-get update
apt-get install manila-share python-pymysql lvm2 nfs-kernel-server -y
```

- Mình sẽ add thêm một ổ cứng để tạo Volume Group trên ổ cứng này.

- Thực hiện tạo volume group
```
pvcreate /dev/sdb
vgcreate manila-volumes /dev/sdb
```

<img src="http://i.imgur.com/0Xav3mA.png">

- Đặt filter cho LVM. Bởi vì mặc định LVM sẽ scan trong thư mục /dev của  các thiết bị block storage device mà bao gồm các volume. Nếu các project sử dụng LVM trên những volume của họ, tool scan sẽ phát hiện những volume đó và cố gắng để cache chúng, do vậy nó có thể gây ra nhiều vấn đề với cả OS và project volume. Cấu hình để LVM chỉ scan ổ có manila-volumes volume group.

Mở file `/etc/lvm/lvm.conf` tìm dòng sau

```
filter = ["a/sdb","r/.*a/"]
```

*Với a là allow còn r là reject với cấu hình này ta sẽ chỉ cho phép scan trên ổ sdb*

- Cấu hình manila-share. Ta mở file /etc/manila/manila.conf và cấu hình như sau

Thêm vào phần [DEFAULT] như sau

```
enabled_share_backends = lvm1  # lvm1 là tên backend
enabled_share_protocols = NFS,CIFS # giao thức cho phép là NFS và CIFS
```

Tạo backend lvm1, ta thêm một phần [lvm1] như sau

```
[lvm1]
share_backend_name = LVM-1 # tên backend
share_driver = manila.share.drivers.lvm.LVMShareDriver # driver sử dụng
driver_handles_share_servers = False # False: driver không xử lý share server
lvm_share_volume_group = manila-volumes  ## tên volume group vừa tạo ở trên
lvm_share_export_ip = 172.16.25.148 ## IP sẽ expose cho các instance kết nối đến. mình sẽ dùng IP external do vậy các instance có thể kết nối đến được
```
- Restart manila-share

```
service manila-share restart
```

- Ta kiểm tra service manila-share đã up chưa bằng lệnh

```
manila service-list
```
<img src="http://i.imgur.com/COhh0vb.png">

- Tạo share-type DHSS = False nếu chưa tạo

```
manila type-create lvm False
```

<img src="http://i.imgur.com/9jrTIxe.png">

- Để chỉ định 1 backend trên share-type nào đó ta thực hiện như sau

```
manila type-key lvm set share_backend_name=LVM-1
```

Kiểm tra các extra-specs

<img src="http://i.imgur.com/KezlTv4.png">

- Tạo một share dùng giao thức NFS kích thước 4GB , sử dụng backend lvm và có tên là `share-lvm-1`


```
manila create nfs 4 --name share-lvm-1 --share-type lvm
```
        - NFS là protocol
        - 4 là kích thước (GB)
        - `--name`: tên của share
        - `--share-type` chọn share-type

- Sau khi tạo share ta cần một đường dẫn để mount trên client

```
root@controller:/home/saphi# manila show share-lvm-1
+-----------------------------+-------------------------------------------------------------------------------------+
| Property                    | Value                                                                               |
+-----------------------------+-------------------------------------------------------------------------------------+
| status                      | available                                                                           |
| share_type_name             | lvm                                                                                 |
| description                 | None                                                                                |
| availability_zone           | nova                                                                                |
| share_network_id            | None                                                                                |
| export_locations            |                                                                                     |
|                             | path = 172.16.25.148:/var/lib/manila/mnt/share-ac9cd421-6f33-492f-8094-2dc21640bcb0 |
|                             | preferred = False                                                                   |
|                             | is_admin_only = False                                                               |
|                             | id = adc5a28e-fa64-464c-ac04-75aaa16911c6                                           |
|                             | share_instance_id = ac9cd421-6f33-492f-8094-2dc21640bcb0                            |
| share_server_id             | None                                                                                |
| host                        | storage@lvm1#lvm-single-pool                                                        |
| access_rules_status         | active                                                                              |
| snapshot_id                 | None                                                                                |
| is_public                   | False                                                                               |
| task_state                  | None                                                                                |
| snapshot_support            | True                                                                                |
| id                          | 7e478fbb-4867-4002-b9b5-34a56d585af0                                                |
| size                        | 4                                                                                   |
| name                        | share-lvm-1                                                                         |
| share_type                  | 79f0d08e-edc9-4409-9902-28cc69ad4d16                                                |
| has_replicas                | False                                                                               |
| replication_type            | None                                                                                |
| created_at                  | 2016-06-05T12:12:35.000000                                                          |
| share_proto                 | NFS                                                                                 |
| consistency_group_id        | None                                                                                |
| source_cgsnapshot_member_id | None                                                                                |
| project_id                  | 2dbc33e520b84b8ab550b649099d7972                                                    |
| metadata                    | {}                                                                                  |
+-----------------------------+-------------------------------------------------------------------------------------+
```

- Để client có thể mount được ta cần cấu hình access-allow. Với LVM sẽ là dựa trên địa chỉ IP, mặc định sẽ có quyền RW ta có thể thay đổi bằng thêm option `--access-level LEVEL` (RW, RO)


```
root@controller:/home/saphi# manila access-allow share-lvm-1 ip 172.16.25.153
+--------------+--------------------------------------+
| Property     | Value                                |
+--------------+--------------------------------------+
| share_id     | 7e478fbb-4867-4002-b9b5-34a56d585af0 |
| access_type  | ip                                   |
| access_to    | 172.16.25.153                        |
| access_level | rw                                   |
| state        | new                                  |
| id           | 272ff935-0b5c-4181-b802-48bf9c4e30d1 |
+--------------+--------------------------------------+
```
*Lưu ý: Các instance khi  mount phải được floating IPs vì LVM sẽ kiểm tra việc được truy cập hay không dựa trên IP này*

- Thực hiện mount trên 1 instance

<img src="http://i.imgur.com/GjsvLNS.png">

- Để delete share

```
manila delete test-share-1
```

Các phần tiếp theo:

[Phần 2: Generic Driver](https://sapham.net/openstack-manila-phan-2-generic-driver/)

[Phần 3: LVM Driver](https://sapham.net/openstack-manila-phan-3-lvm-driver/)

[Phần 4: GlusterFS Native Driver](https://sapham.net/openstack-manila-phan-4-native-glusterfs-driver/)