---
layout: post
title: SR-IOV trong Openstack Phần 1
image: "/content/images/2016/06/openstack-4.jpg"
date: '2018-09-01 16:17:00'
tags:
- openstack
- virtualization
- nova
- neutron
- openvswitch
- linuxbridge
---

###SR-IOV trong Openstack Phần 1

####SR-IOV là gì?

SR-IOV viết tắt của `Single Root I/O Virtualization` là một kỹ thuật mà cho phép một thiết bị PCI chia thành nhiều thiết bị riêng biệt gồm có Physical Function (PF) và một hoặc nhiều Virtual Function (VF). SR-IOV cung cấp một cách chuẩn cho một thiết bị vào ra vật lý để present chính nó để PCIe bus như nhiều thiết bị ảo.

Trong khi các PF có tất cả các tính năng của PCIe thì VFs là các chức năng nhẹ mà không đủ các cấu hình tài nguyên. Cấu hình các VFs và quản lý được thực hiện thông qua PF vì vậy chúng nó thể chỉ tập tập trung vào việc di chuyển dữ liệu. Điều này rất quan trọng để lưu ý là tổng bandwidth sẵn có với PF được chia sẻ cho tất cả các VF mà tương tác với nó.

SR-IOV yêu cầu sự hỗ trợ từ BIOS , OS hay Hypervisor và phần cứng.

- Danh sách OS hỗ trợ

```
Windows Server 2012
Windows Server 2012 R2
Windows Server 2008* R2
Windows Server 2008
Linux* 2.6.30 kernel or later
Red Hat Enterprise Linux 6.0* and later
SUSE Linux Enterprise Server 11* SP1 and later

```
- Danh sách Hypervisor hỗ trợ

```
Microsoft Hyper-V* (Windows Server 2012*)
VMware Sphere* 5.1
Xen Hypervisor*
KVM* (Kernel Based Virtual Machine)
```

Về lý thuyết mỗi một thiết bị SR-IOV có tối đa 256 VF

Ví dụ

- 1 NIC card Gigabit Ethernet 4 cổng hỗ trợ SR-IOV tức là 4 devices, với mỗi device ta có 256 VFs do vậy tổng VF có thể là 1024 VFs trong trường hợp này mỗi VF về cơ bản sẽ đại diện cho 1 cổng Ethernet duy nhất

- 1 HBA (Host bus adapter) dual-port thể hiện là 1 device dual-port. Với mỗi device ta có 256 VFs kết quả ta sẽ có 512 HBA port trải trên 256 dual-port

Lý thuyết là vậy nhưng thực tế thì giới hạn này nhỏ hơn nhiều. Hiện tại 64 VFs được xem là giới hạn cho hầu hết các PCIe device.

Trong trường hợp của network, SR-IOV cho phép một card mạng vật lý thể hiện như nhiều card PCIe network. Mỗi một port network vật lý trên NIC được thể hiện như một Physical Function và mỗi PF có thể tương tác với số các VF được cấu hình.

Cấp phát một VF tới một máy ảo cho phép network traffic tới VM không cần qua một lớp software của hypervisor do vậy data flow trực tiếp giữa VF và VM.

Bởi vì network traffic hoàn toàn không cần qua một lớp software của hypervisor bao gồm các switch ảo do vậy card network vật lý chịu trách nhiệm quản lý luồng traffic.

<img src="http://i.imgur.com/OhsJ1pm.png">
*Nguồn: http://redhatstackblog.redhat.com/*
####SR-IOV trong Openstack

Theo truyền thống, một neutron port được gắn tới tới một bridge ảo (OpenVswitch, LinuxBridge,..) trên node Compute. Với sự xuất hiện của SR-IOV trong network, nó có thể kết hợp một neutron port với một Virtual Function trên network adapter. Vì vậy các bridge ảot trên Compute node sẽ không cần thiết.

Khi một packet đến một port vật lý, nó sẽ được đặt vào trong pool các VF dựa trên địa chỉ MAC hoặc VLAN tag. Hypervisor sẽ không tham gia vào việc xử lý các packet, vì vậy loại bỏ khả năng ngẽn cổ chai (bottlenecks).  VM sử dụng các SR-IOV port và VM sử dụng port thuần túy (liên kết với OpenVswitch, LinuxBridge,..) có thể giao tiếp với nhau thông qua network miễn là được cấu hình phù hợp (flat, VLAN).

Trong khi Ethernet là một công nghệ hầu hết được triển khai ngày nay. Ta cũng có thể sử dụng SR-IOV với các port sử dụng các công nghệ khác như InfiniBand (IB). Tuy nhiện hiện tại SR-IOV Neutron ML2 driver chỉ hỗ trợ các port Ethernet.


####Tại sao SR-IOV và Openstack

Lý do chính để sử dụng SR-IOV là hiệu năng (throughput, delay,..) cho mỗi VM. Một use case phổ biết là Network Functions Virtualization (NFVs).