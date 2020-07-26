---
layout: post
title: 'Openstack manila phần 5: NFS-Ganesha Driver'
image: "/content/images/2016/06/openstack-3.jpg"
date: '2017-02-02 17:32:00'
tags:
- openstack
- glusterfs
- manila
- shared-file-system
- storage
- nfs-ganesha
---

#####Openstack manila phần 5: NFS-Ganesha Driver

GlusterFS Driver với NFS-Ganesha làm gateway

Sau khi cấu hình GlusterFS làm backend cho manila ta đã thấy một số nhược điểm

- Là dạng Volume mapping do vậy ta sẽ phải thực hiện tạo các volume trước khi tạo share.
- Nếu xem log các bạn có thể thấy để tạo một share thì manila-host sẽ phải SSH rất nhiều lần vào glusterfs-server(bằng số volume). Tham khảo log khi tạo share [manila-share.log](http://pastebin.com/vLMCPjmm)

Có một lựa chọn là ta có thể sử dụng NFS-Ganesha làm gateway cho GlusterFS Server sẽ có những ưu điểm sau

- Là dạng directory mapping, mỗi 1 share sẽ tạo ra 1 directory trên volume của glusterfs
- Dễ dàng giới hạn kích thước share (Không cần tạo nhiều volume có kích thước khác nhau trên glusterfs do vậy không cần cấu hình volume pattern)


Ta sẽ cần thêm 1 node cài nfs-ganesha server và mình sẽ sử dụng CentOS 7 để cài.
Mô hình cài đặt sẽ như sau

<img src="http://i.imgur.com/iiuSAJh.png">

######Cài đặt

- Cài đặt NFS-ganesha

download repo nfs-ganesha và glusterfs

```
curl -o /etc/yum.repos.d/nfs-ganesha.repo http://download.gluster.org/pub/gluster/glusterfs/nfs-ganesha/2.3.0/EPEL.repo/nfs-ganesha.repo
curl -o /etc/yum.repos.d/glusterfs-epel.repo http://download.gluster.org/pub/gluster/glusterfs/3.7/3.7.11/EPEL.repo/glusterfs-epel.repo
```
update repo

```
yum update
```

Cài đặt nfs-ganesha và glusterfs-client

```
yum -y install epel-release
yum -y install glusterfs-ganesha
```

Ta gluster server mình sẽ tạo một volume tên là `vol` để cấu hình trên ganesha
Ta có thể disable nfs trên volume này

```
gluster volume set vol nfs.disbale on
```

File cấu hình `/etc/ganesha/ganesha.conf` sẽ như sau

```
EXPORT
{
	# Export Id (bắt buộc, mỗi export có một đường Id)
	Export_Id = 78;

	# đường dẫn export (bắt buộc)
	Path = /shared;

	# đường dẫn Pseudo  (yêu cầu cho NFS v4)
	Pseudo = /shared;

	# Required for access (default is None)
	# Could use CLIENT blocks instead
	Access_Type = RW; # read/write

	# Exporting FSAL
	FSAL {
		Name = GLUSTER; # File system abstract layer là GLUSTER
		Hostname = "10.0.0.28"; # Ip, hostname của 1 node trong cụm glusterfs
		Volume =vol; # ten volume
	}
}

```

- Khởi động lại nfs-ganesha

```
service nfs-ganesha restart
```

- Kiểm tra export

```
showmount -e localhost
```

<img src="http://i.imgur.com/J2nwYAj.png">

- Cấu hình trên manila-share ở đây mình tiếp tục dùng node compute1 để chạy manila-share
mở file `/etc/manila/manila.conf` thêm đoạn sau

```
[ganesha]
share_backend_name = ganesha #tên backend
glusterfs_nfs_server_type = Ganesha # sử dụng ganesha server
glusterfs_target=root@10.0.0.28:/vol # user, IP(hostmame) , volume name của Gluster được cấu hình trên gluster
glusterfs_server_password=saphi # password để login vào gluster server
share_driver = manila.share.drivers.glusterfs.GlusterfsShareDriver # driver
ganesha_service_name = nfs-ganesha # tên service của ganesha trên ganesha-server
driver_handles_share_servers = False
glusterfs_ganesha_server_ip=172.16.25.146 # IP của ganesha server
glusterfs_ganesha_server_username=root # user để ssh
glusterfs_ganesha_server_password=saphi # password
```

sau đó enable backend này lên

```
enabled_share_backends = generic,ganesha
enabled_share_protocols = NFS,CIFS,GLUSTERFS
```
*Lưu ý:*

- Tại sao cần manila host cần ssh vào gluster? Bởi vì manila sẽ truy cập vào và đặt các quota cũng như các export trên volume đó. Ta có thể kiểm tra cmd_history trên glusterfs-server


<img src="http://i.imgur.com/JAfuv1N.png">




- Kiểm tra service manila

```
manila service-list
```

<img src="http://i.imgur.com/FImqfj7.png">

- Tạo share type ta cần disable snapshot vì đây là dạng directory mapping do vậy ko hỗ trợ snapshot

```

manila type-create ganesha False
manila type-key ganesha set share_backend_name=ganesha
manila type-key ganesha set snapshot_support=False

```
<img src="http://i.imgur.com/uLwJyl4.png">


- Tạo một share có tên `share-2` dung lượng 2GB trên backend `ganesha`

```
manila create nfs 2 --name share-2 --share-type ganesha
```
<img src="http://i.imgur.com/Rp88dFq.png">


- Set access allow theo IP

<img src="http://i.imgur.com/0wVD1nU.png">

Khi tạo một access ta thử kiểm tra file export share trên ganesha server tại đường dẫn `/etc/ganesha/export.d/`sẽ như sau

<img src="http://i.imgur.com/D90X63q.png">

- Kiểm tra đường dẫn mount

<img src="http://i.imgur.com/RoSg5Pq.png">

Ta thấy đường dẫn mount yêu cầu thêm `access-id` chính là ID của access ta tạo ở trên.

- Thực hiện mount kiểm tra trên client

<img src="http://i.imgur.com/1HBMUAj.png">

- Ta thấy mỗi một share được tạo ra sẽ là một export trên nfs-ganesha server và ứng với một share là một directory mới được tạo ra trên share server. Và các export được set đúng quota như share..