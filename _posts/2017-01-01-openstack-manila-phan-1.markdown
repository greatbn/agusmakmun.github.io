---
layout: post
title: Openstack Manila phần 1
image: "/content/images/2016/06/openstack-2.jpg"
date: '2017-01-01 16:40:00'
tags:
- openstack
- manila
- shared-file-system
- storage
---

###Tổng quan dịch vụ Shared File System

Dịch vụ Openstack Shared File Systems (manila) cung cấp file storage cho một máy ảo. Dịch vụ Shared File Systems cung cấp một cơ sở hạ tầng để quản lý và cung cấp các file chia sẻ. Dịch vụ cũng cho phép quản lý các kiểu chia sẻ cũng như các share snapshot nếu driver hỗ trợ chúng.

Dịch vụ Shared File System có các thành phần sau.

- manila-api: xác thực và route các request tới dịch vụ. Nó hỗ trợ các API Openstack.

- manila-scheduler: Lập lịch và chuyển các requét với share service thích hợp. Scheduler sử dụng các cấu hình lọc và weigh để route các requét. Filter Scheduler là  mặc định và cho phép filter dựa trên: Khả năng lưu trữ (Capacity), Các zone (Availability Zone), Kiểu share (share-type) và các filter custom.

- manila-share: quản lý các backend cung cấp shared file system. Một tiến trình manila-share có thể chạy trên 1 hoặc 2 node.

####Share, Snapshot,share-network

Là các resource cơ bản đưa ra bởi dịch vụ Shared File Systems là shares, snapshots, và share network

- share: là một đơn vụ của storage với một giao thức, một kích thước và một danh sách được phép truy cập. Tất cả các share tồn tại trên một backend. Một vài share tương tác với share network và share server. Các giao thức chính được hỗ trợ là NFS và CIFS, ngoài ra còn có các giao thức khác được hỗ trợ như GLUSTERFS, ..

- snapshots:  Snapshot là một thời điểm copy của một share. Snapshot có thể được sử dụng để tạo một share mới. Share không thể bị xóa nếu có các snapshot tạo ra trên share đó(ta phải xóa hết snapshot của share đó  rồi mới xóa được share)

- share network: Một share network là một đối được được định nghĩa bởi một project(tenant) mà báo cho Manila về security và cấu hình mạng cho một nhóm của các share. Share network chỉ thích hợp cho backend quản lý share server.  Một share network bao gồm các dịch vụ bảo mật ,network và subnet.

####Cài đặt manila

Với manila ta có thể cấu hình trên một node hoặc nhiều node. Mình sẽ cài đặt trên nhiều node
- Các thành phần manila-api và manila-scheduler ta sẽ cài đặt trên node controller
- Và manila-share sẽ cài đặt trên một node riêng.

Khi cài đặt manila sẽ cho ta hai lựa chọn. Đó là cấu hình có hoặc không có việc xử lý của share server. Và các lựa chọn này dựa trên driver ta sử dụng.

- Lựa chọn 1: Cấu hình không có việc xử lý của share server. Trong mode này dịch vụ không can thiệp tới network. Người vận hành phải chắc chắn việc kết nối nữa các instance và Share Server. Với lựa chọn này ta sử dụng driver LVM, GLUSTERFS,...

- Lựa chọn 2: Triển khai dịch vụ vói sự hỗ trợ của driver cho việc quản lý các share. Trong mode này, các service yêu càu là nova, neutron và cinder cho việc quản lý share server. Thông tin được sử dụng cho việc tạo share server là share network. Option này sử dụng generic driver với việc sử lý các  share server và đòi hỏi phải được gắn vào mạng `selfservice` tới một router.

*Note: Với lựa chọn 2 ta có thể hình dung đơn giản như sau. Để sử dụng option này ta cần phải tạo ra một share-network dựa trên cấu hình neutron.
Share network này sẽ được gắn với router của tenant và thông với tenant-network.

Sau khi có một yêu cầu tạo share. Khi Yêu cầu này sẽ được chuyển tới 1 backend có Cấu hình `Generic Driver` trên node cài manila-share. manila-share được cấu hình sử dụng nova để tạo ra một instance (image được cài sẵn nfs-server), đồng thời 1 volume tạo ra trên cinder có kích thước bằng kích thước của share tạo ra volume này gắn với instance trên. Và các dữ liệu share sẽ được lưu tại volume này*

Mô hình mạng sẽ như sau

<img src="http://i.imgur.com/TmiBQi7.png">

Notes: Mình mới test thử trên Openstack L2-Agent sử dụng OpenvSwitch chưa test được trên LinuxBridge


Kết luận: Openstack manila là một trong số những project quan trọng của Openstack. Các phần tiếp theo mình sẽ đi cấu hình và trải nghiệm với một số Driver mà Manila hỗ trợ như LVM, GLUSTERFS, GENERIC, ...


Các phần tiếp theo:

[Phần 2: Generic Driver](https://sapham.net/openstack-manila-phan-2-generic-driver/)

[Phần 3: LVM Driver](https://sapham.net/openstack-manila-phan-3-lvm-driver/)

[Phần 4: GlusterFS Native Driver](https://sapham.net/openstack-manila-phan-4-native-glusterfs-driver/)