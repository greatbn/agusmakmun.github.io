---
layout: post
title: Tìm hiểu hugepages trong virtualization
date: '2016-06-01 17:08:01'
tags:
- openstack
- linux
- virtualization
---

#####Tìm hiểu hugepages trong virtualization

#####1.Pagesize

Một Bộ nhớ chính được phân thành một tập các vùng liên tục được gọi là pages. Mặc định kích thước một pages này là 4K bytes. Do vậy khi dung lượng bộ nhớ tăng lên số lượng pages cũng tăng lên rất nhanh.

Ví dụ: Một bộ nhớ  RAM 10GB  ta sẽ có 2.6 triệu pages.


Các tiến trình thì được CPU cấp phát cho vùng nhớ có địa chỉ nhớ ảo.
Khi tiến trình muốn truy cập vào dữ liệu trên bộ nhớ chính nó sẽ sử dụng PageTable để thực hiện việc ánh xạ địa chỉ ảo này sang địa chỉ vật lý.

Việc tìm kiếm và ánh xạ mất rất nhiều thời gian do phải tìm từng pages một. Do vậy người ta đã thiết kế một bộ nhớ cache để  lưu trữ các pages được truy cập gần đây nhất gọi là Translation Lookaside Buffer (TLB) nhưng bộ cache này có kích thước rất nhỏ nên vẫn không giải quyết được vấn đề về truy vấn vào bộ nhớ ram.

Do vậy Hugepages đã được phát minh. Đúng với tên gọi, nó là các pages có kích thước lớn.

Ở một số kiến trúc khác nhau sẽ có kích thước pages khác nhau và có thể có nhiều kích thước trên cùng một kiến trúc. Ví dụ x86_64 hỗ trợ 2 kích thước pages là 2MB và 1GB

Làm một phép tính đơn giản ta sẽ thấy nếu ta có bộ nhớ ram 10GB và sử dụng Hugepages size là 2MB thì số pages lúc này sẽ là 5000 pages (ban đầu là 2.6 triệu pages)

Vậy việc tìm kiếm và ánh xạ địa chỉ ảo và địa chỉ vật lý sẽ nhanh hơn do tìm kiếm nhanh hơn

#####2.Paged out

Khi sử dụng TLB làm bộ nhớ đệm. Vì kích thước nhỏ nên khi ta truy xuất vào để ánh xạ địa chỉ vật lý sẽ xảy ra 2 trường hợp. Một là có tồn tại trong TLB và từ đó lấy được địa chỉ vật lý (TLB Hit). Hai là không có trong TLB (TLB miss) do đó CPU sẽ thực hiện tìm kiếm trên PageTable. Và update giá trị ánh xạ mới vào TLB. Nếu bộ nhớ TLB đầy các entry không được sử dụng lâu nhất sẽ bị đẩy ra vùng nhớ khác (swap) ta gọi đó là Paged out.

Khi sử dụng Hugepages các entry này sẽ luôn luôn tồn tại trong Hugepages do vậy không có hiện tượng Paged out.

#####3.Kết luận: 

Hugepages sẽ có hai ưu điểm làm tăng tốc hiệu năng của ứng dụng sử dụng nó và có 2 điểm chính cần lưu ý:

- Pagesize lớn hơn mặc định (4KB): Có thể là 2MB , 1GB , ..

- Các entry bị locked trong Hugepages và không thể paged out

#####4.Cấu hình trên Ubuntu 14.04 + KVM

Để thực hiện cấu hình cho các VMs trên Hypervisor KVM sử  dụng hugepages ta cần enable hugepages bằng việc khai báo các hugepages trong /etc/sysctl.conf

Ví dụ tôi muốn dùng 2GB cho Hugepages ta sẽ cần dùng 1000 pages

Thêm dòng sau

```
vm.nr_hugepages = 1000
```

Gõ `sysctl -p` để áp dụng cấu hình

Kiểm tra meminfo

`cat /proc/meminfo | grep Huge`

Kết quả như sau

<img src="http://i.imgur.com/KVDe4uW.png">

- Hugepages_Total: là tổng số pages

- Hugepages_Free: Số pages chưa sử dụng

- Hugepages_Rsvd: Số pages được cấp phát từ pool tạo ra nhưng vẫn chưa được cấp phát. (Ngắn gọn là để dành)

- Hugepages_Surp: Số hugepages trong pool (dư thừa)

- Hugepagesize: Kích thước Hugepages


Bước tiếp theo ta enable QEMU-KVM sử dụng Hugepages bằng sửa `KVM_HUGEPAGES=1` trong file `/etc/default/qemu-kvm`


Restart lại libvirt `service libvirt-bin restart`



#####5.Launch VMs sử dụng hugepages

Mình đang có chạy 1 VM cirros với RAM là 512MB

Mình sẽ edit KVM xml file bằng virsh

`virsh edit cirros`

thêm đoạn sau

```
<memoryBacking>
<hugepages/>
</memoryBacking>

```

Restart vm

```
virsh destroy cirros
virsh start cirros

```


Kiểm tra meminfo `cat /proc/meminfo | grep Huge`

<img src="http://i.imgur.com/WGz2pin.png">

Số Hugepages_Free lúc này là 744 =>  đã sử dụng 256 pages => 512MB => Bằng số ram của VM cirros
