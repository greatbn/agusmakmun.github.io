---
layout: post
title: Windows Container và Docker một vài điều cần phải biết
image: "/content/images/2016/09/1805-NANO-SERVER_6373D571.png"
date: '2016-09-05 16:05:16'
tags:
- virtualization
- docker
- windows
- container
- hyper-v
---

Microsoft Ignite sắp tới, Windows Server 2016 sẽ được phát hành và Windows Container sẽ sẵn sàng cho môi trường Production. Xin nhắc lại tôi không nói về Docker trong một VM linux chạy trên Windows mà là Docker chạy trực tiếp như một Windows Service. Đây là một vài điều ta cần biết trong quá trình chuẩn bị

####1. Windows Server container hay Hyper-V container

Windows có hai mô hình để chạy container

**Windows Server Container** theo cấu trúc hiện tại của Docker trên Linux: các container chia sẻ chung kernel từ hệ điều hành của host, do vậy nó sẽ nhẹ và nhanh. Khi ta chạy một process bên trong một container, process đấy thực sự là chạy trên host và ta có thể thấy trong Task Manager.

**Hyper-V Container** chạy trong một máy ảo rất nhỏ bên trên của host, do vậy các container có kernel của nó. Khi ta chạy một process bên trong Hyper-V container, host không biết về các process này. Hyper-V container sẽ cung cấp một cách cô lập cao hơn, lớp VM được tối thiểu do vậy hiệu năng vẫn tốt. 
Với Windows Server 2016 ta có thể chọn giữa hai runtime này, hiện tại Windows 10 chỉ chạy trên Hyper-V container.

####2. Windows bên trong container 

Hiện tại, ta không thể chạy các tiến trình Linux trực tiếp trên windows, vì vậy bạn không thể chạy các tiến trình Linux trong Windows Container. Trong cả hai trường hợp Windows Container và Hyper-V container đều như vậy. Kernel mà container thấy được là Windows vì vậy nó chỉ có thể chạy các tiến trình của Windows.

####3. Hybrid Swarm

Swarm Mode được đóng sẵn trong Docker Engine từ phiên bản 1.12, vì vậy bất kể host nào đang chạy Docker có thể join vào một swarm. Chức năng này cũng có trong Docker Engine cho Windows. Có nghĩa là ta có các host Linux và Windows trong cùng swarm cluster. Các container chạy trên các kernel khác nhau có thể kết nối qua nhau trên cùng một Docker network.

Hybrid Docker swarm cho ta một lộ trình đẻ Dockerize các ứng dụng Windows, phá vỡ rào cản về sự tích hợp chúng với các phần mềm từ thế giới của Linux.

####4. License 

Các phiên bản của Windows thường ta sẽ phải trả phí cho license và các image của Windows cũng vậy. Nhưng hiện tại tôi chưa biết cách quản lý như thế nào từ M$. Windows Server 2016 sẽ được phát hành tại Ignite, ta sẽ hiểu hơn về vụ License này.

Có thể M$ sẽ phân phối các image này thông qua Docker Store - một registry image để thuơng mại, cho phép các nhà phát hành tính phí các image có bản quyền. 
