---
layout: post
title: K8S-101 - Các khái niệm trong Kubernetes
image: ''
date: '2017-03-11 15:46:38'
tags:
- kubernetes
- container
- k8s
---

#### Kubernetes là gì?

Kubernetes là một nền tảng mã nguồn mở giúp tự động triển khai, scale và vận hành các ứng dụng container trên cụm các host, cung cấp cơ sở hạ tầng tập trung container

Với kubernetes ta có thể giải quyết các nhu cầu của khác hàng nhanh chóng và rất hiệu quả

- Deploy ứng dụng nhanh

- Scale ứng dụng mà không ảnh hưởng tới các thành phần khác

- Liên tục có những tính năng mới

- Tối ưu việc sử dụng tài nguyên


#### Các khái niệm
**Cluster**

Một cluster là một tập các máy vật lý hay máy ảo hoặc những tài nguyên cơ sở hạ tầng khác được sử dụng bởi Kubernetes để chạy các ứng dụng 

**Node**

Một node là một máy ảo hoặc máy vật lý chạy Kubernetes, nơi mà các pod có thể được schedule tới đó.

**Pod**

Một pod là một nhóm các container và volume có liên quan tới nhau

**Label**

Một label là một cặp key/value gắn với một tài nguyên (resource) như một pod, để truyền tải các thuộc tính mà người dùng định nghĩa ra. Các label có thể được sử dụng để tổ chức và lựa chọn các tập con của nhiều resource khác nhau.

**Selector**

Một Selector là một biểu thức mà phù hợp với label để xác định các resource liên quan. Ví dụ: Những pod nào là backend của một load balancer

**Replication Controller**

Một replication controller đảm bảo rằng số lượng các pod replicas đã định nghĩa luôn luôn chạy tại bất kì thời điểm nào. Nhiệm vụ là xử lý việc tạo lại những pod này khi máy chủ đó bị reboot hoặc lỗi.

**Service**

Một service định nghĩa một tập các pod và cách để truy cập vào chúng, ví dụ như một địa chỉ IP hoặc 1 tên miền

**Volume**

Một volume đơn giản là một thư mục, có thể có dữ liệu bên trong, nó được truy cập bởi một container. 

**Secret**

Một secret chứa dữ liệu nhạy cảm, như token. Nó sẽ nằm bên trong container khi có yêu cầu. 

**Name**

Tên của resource 

**Namespace**

Một namespace như một prefix cho tên của một resource. Namespace giúp cô lập các project, team hay các khách hàng trên cùng 1 cluster với nhau. Namespace cũng tránh việc trùng lặp tên giữa các team.

*Annotation*

Một cặp key/value - có thể lưu giữ lượng dữ liệu lớn hơn (khi so  sánh với label) và có thể người dùng không đọc hiểu được dữ liệu đó. 