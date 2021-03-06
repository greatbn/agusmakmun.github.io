---
layout: post
title: Rootwrap trong Openstack dùng để làm gì?
image: "/content/images/2016/08/openstack.jpg"
date: '2016-08-20 13:37:04'
tags:
- openstack
- virtualization
- nova
- rootwrap
---

Khi quản trị một hệ thống Openstack ta thường thấy có một thư mục `rootwrap.d` và file config `rootwrap.conf` tại thư mục của các project như neutron, nova, cinder, ...
Vậy thành phần này có nhiệm vụ gì trong các hoạt động của Openstack. Ta sẽ xét trong bài này. 

<img src="http://i.imgur.com/QXn4lbD.png">
*Cấu trúc thư mục cấu hình cho nova*

Mục đích của rootwrap là cho phép một user không có đặc quyền để thực hiện một vài hành động của một user `root` theo một cách an toàn nhất. 

Ở một số phiên bản ban đầu, Nova sử dụng file `sudoers` để liệt kê các lệnh mà user nova được phép chạy ở quyền root, ta chỉ cần thêm  `sudo` vào trước mỗi câu lệnh. 
Khi số lượng các câu lệnh tăng lênh việc quản trị trở nên khó khăn hơn. Và nó cũng không cho phép thực hiện lọc các lệnh theo các tham số đầu vào. Rootwrap sinh ra để giải quyết các vấn đề đó. 

#####Rootwrap làm việc như thế nào?

Thay vì việc gọi câu lệnh `sudo make me a sandwich`. Nova sẽ thực hiện câu lệnh `sudo nova-rootwrap /etc/nova/rootwrap.conf make me a sandwich`. Một file `sudoers` thông thường cho phép user nova chạy `nova-rootwrap` ở quyền `root`. 
Nova-rootwrap sẽ tìm các filter mà user định nghĩa trong thư mục `rootwrap.d` và file cấu hình của nó và thực hiện load các `command filter` từ chúng.  
Sau đó nó sẽ kiểm tra xem câu lệnh có match với những filter này không, nếu match nó sẽ thực hiện command vơi quyền root. Mặt khác command đó sẽ bị loại bỏ. 

Ta sẽ xem xét một số cấu hình cho rootwrap. Tôi sẽ lấy nova làm ví dụ các service khác cũng tuơng tự. 

Để cấu hình rootwrap trên nova. Ta cần cấu hình tham số `rootwrap_config ` và trỏ tới file cấu hình của rootwrap. 
Ví dụ tôi có 

```
rootwrap_config = /etc/nova/rootwrap.conf
```

File cấu hình này sẽ phù hợp với một entry trong sudoers. Để hiểu rõ hơn tôi sẽ mở file sudoers của nova tại đường dẫn 
```
/etc/sudoers.d/nova_sudoers
```
sẽ có nội dung như sau 

```
Defaults:nova !requiretty

nova ALL = (root) NOPASSWD: /usr/bin/nova-rootwrap /etc/nova/rootwrap.conf *
```

Tiếp tục ta sẽ mở file cấu hình của rootwrap 

```
/etc/nova/rootwrap.conf
```
Nội dung như sau 

<img src="http://i.imgur.com/zusrpVg.png">

Ở section `DEFAULT`

- tham số `filters_path` chỉ ra đường dẫn tới thư mục định nghĩa các filter. Nova cung cấp các file filter tại `/usr/shar/nova/rootwrap` và các filter mở rộng tại `/etc/nova/rootwrap.d`

- tham số `exec_dirs` là danh sách các thư mục để nova tìm file thực thi ở đây là `nova-rootwrap`. Nova sẽ thực hiện tìm kiếm file thực thi lần lượt tại các thư mục này khi phát hiện file thực thi thì sẽ dừng lại.
- `use_syslog`: enable syslog. Mặc định là False

Các file filter sẽ có dạng `*.filters` và nằm trong thư mục `/etc/nova/rootwrap.d`

<img src="http://i.imgur.com/lct4Z4w.png">

Với nova ta chỉ có một file là `api-metadata.filters`


Nội dung của file này như sau

```
[Filters]
iptables-save: CommandFilter, iptables-save, root
ip6tables-save: CommandFilter, ip6tables-save, root
iptables-restore: CommandFilter, iptables-restore, root
ip6tables-restore: CommandFilter, ip6tables-restore, root

```

Với `CommandFilter` nova chỉ kiểm tra command thực thi
Ví dụ ở trên ta có: 
```
iptables-save: CommandFilter, iptables-save, root
```
sẽ cho phép thực hiện lệnh `iptables-save` với quyền root và tất cả các parameter của nó. 

Ngoài `CommandFilter` ta còn một số Filter khác như 

- RegExpFilter
- PathFilter
- EnvFilter
- ReadFileFilter
- KillFilter
- IpFilter
- IpNetnsExecFilter

Tham khảo thêm tại [Openstack rootwrap](https://wiki.openstack.org/wiki/Rootwrap)