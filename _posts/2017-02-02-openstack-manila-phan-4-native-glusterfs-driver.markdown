---
layout: post
title: 'Openstack manila phần 4: Native GlusterFS Driver'
image: "/content/images/2016/06/openstack-2.jpg"
date: '2017-02-02 12:35:00'
tags:
- openstack
- glusterfs
- manila
- shared-file-system
- storage
---

######Openstack manila phần 4: Native GlusterFS Driver

Tiếp tục loạt bài về Openstack Manila hôm nay tôi sẽ cấu hình backend sử dụng GlusterFS

- Yêu cầu phiên bản GlusterFS >= 3.6. Với glusterfs nếu cluster của bạn không hỗ trợ snapshot thì trên manila cũng sẽ mất đi tính năng này. Để cấu hình snapshot ta sẽ cấu hình Thin Provision theo bài hướng dẫn link

- Với bài lab của mình có 2 node và chạy kiểu replicate. Mình sẽ tạo các thinly provisioned và tạo volume trên đó.

Mô hình cài đặt

<img src="http://i.imgur.com/J4NlcLL.png">

Cài đặt glusterfs-v3.7

```
add-apt-repository ppa:gluster/glusterfs-3.7 -y
apt-get update
apt-get install glusterfs-server -y

```

Tham khảo script tạo thin LV và gluster volume

Script tạo thinly provisioned chạy trên 2 node
```
apt-get install xfsprogs -y
pvcreate /dev/sdb
vgcreate myVG /dev/sdb
lvcreate -L 8G -T myVG/thinpool
for ((i = 1;i<= 5; i++ ))
do
mkdir -p /manila/manila-"$i"
for (( j = 1; j<= 5; j++))
do
lvcreate -V "${i}"Gb -T myVG/thinpool -n vol-"$i"-"$j"
mkfs.xfs /dev/myVG/vol-"$i"-"$j"
mkdir -p /manila/manila-"$i"/manila-"$j"
mount /dev/myVG/vol-"$i"-"$j" /manila/manila-"$i"/manila-"$j"
echo "/dev/myVG/vol-"$i"-"$j" /manila/manila-"$i"/manila-"$j" xfs 0 2" >> /etc/fstab
done
done
```


Tạo gluster volume chỉ cần chạy trên 1 node

```
for (( i= 1 ; i <= 5; i++))
do
for(( j= 1; j<=5 ;j++))
do
gluster volume create manila-"$i"-"$j" replica 2 glusterfs-1:/manila/manila-"$i"/manila-"$j"/br glusterfs-2:/manila/manila-"$i"/manila-"$j"/br
gluster volume start manila-"$i"-"$j"
done
done

```

- Với backend là GlusterFS việc quản lý truy cập vào share file trên manila sẽ thông qua certificate do vậy ta cần cấu hình SSL cho client và server.

Tạo key và CA trên server. Mình sẽ copy key và ca tới tất cả server glusterfs
```
cd /etc/ssl
openssl genrsa -out glusterfs.key 1024
openssl req -new -x509 -key glusterfs.key -subj /CN=saphi -out glusterfs.pem
cp glusterfs.pem glusterfs.ca
```

Ở đây ta tạo CA có `CN=saphi` trên manila ta sẽ tạo access-allow cho CA của `saphi`



- Cấu hình GlusterFS tương tự như LVM mình sẽ sử dụng node compute để cài manila-share.




Cấu hình manila-share tại file `/etc/manila/manila.conf`

Enable backend glusterfsnative và protocol GLUSTERFS tại mục [DEFAULT]

```
enabled_share_backends= glusterfsnative #glusterfsnative là tên phần backend
enabled_share_protocols=NFS,CIFS,GLUSTERFS #giao thức cho phép

```
Cấu hình backend glusterfsnative như sau

```
[glusterfsnative]
share_backend_name = glusterfsnative # tên backend
glusterfs_servers = root@glusterfs1 # khai báo user,IP của gluster server, nếu có nhiều server glusterfs cách nhau bằng dấu phẩy
glusterfs_server_password = saphi # password để manila-host ssh vào glusterfs server
glusterfs_volume_pattern = manila-#{size}-.* # pattern volume trên manila khi tạo share mapping với volume trên glusterfs
share_driver = manila.share.drivers.glusterfs.glusterfs_native.GlusterfsNativeShareDriver # share driver sử dụng
driver_handles_share_servers = False  # False: không sử dụng driver xử lý share server
 
```

Nói thêm về volume-pattern:

Với script ở trên mình đã tạo các volume có mẫu `manila-$i-$j` với $i là size của volume và $j là số thứ tự. Do vậy ta sẽ cấu hình mẫu để các share manila khi tạo ra sẽ mapping với các volume trên glusterfs.

- Sau khi cấu hình xong ta khởi động lại manila-share

```
service manila-share restart
```

- Kiểm tra service manila sẽ có thêm service cho glusterfsnative

<img src="http://i.imgur.com/wjUPlEq.png">

- Trên node `controller` cần phải enable protocol `GLUSTERFS` vì mặc định chỉ enable `NFS` và `CIFS`

thêm dòng sau vào mục [DEFAULT] trong `/etc/manila/manila.conf`

```
enabled_share_protocols=NFS,CIFS,GLUSTERFS
```
sau đó restart lại manila

```
service manila-api restart
service manila-scheduler restart
```

- Tạo share-type

```
manila type-create glusterfsnative False # tạo type tên là glusterfsnative DHSS = False
manila type-key glusterfsnative set share_backend_name=glusterfsnative # khai báo backend cho type này
```

<img src="http://i.imgur.com/qhm59tZ.png">

- Tạo share

```
manila create glusterfs 2 --name gluster-1 --share-type glusterfsnative
```
<img src="http://i.imgur.com/dGb2Fkb.png">

- Tạo access-allow cho CA có `CN=saphi`

<img src="http://i.imgur.com/fB1RSH7.png">

Ta thấy access-type ở đây là `cert`

- Kiểm tra đường dẫn mount path

<img src="http://i.imgur.com/CssIQzA.png">

- Thực hiện mount trên client

Client cần cài gói glusterfs-client

```
add-apt-repository ppa:gluster/glusterfs-3.7 -y
apt-get update
apt-get install glusterfs-client -y
```

Client cần  CA có `CN=saphi` và server phải biết CA này tôi sẽ copy key và CA trên server về client sau đó thực hiện mount

```
mount -t glusterfs glusterfs1:/manila-2-1 /mnt
```

<img src="http://i.imgur.com/lexNgpx.png">

Các phần tiếp theo:

[Phần 2: Generic Driver](https://sapham.net/openstack-manila-phan-2-generic-driver/)

[Phần 3: LVM Driver](https://sapham.net/openstack-manila-phan-3-lvm-driver/)

[Phần 4: GlusterFS Native Driver](https://sapham.net/openstack-manila-phan-4-native-glusterfs-driver/)