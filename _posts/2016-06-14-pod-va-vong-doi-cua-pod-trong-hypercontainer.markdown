---
layout: post
title: Pod và vòng đời của Pod trong Hypercontainer
image: "/content/images/2016/06/1376506597_peas-in-a-pod.jpg"
date: '2016-06-14 14:19:25'
tags:
- hypercontainer
- kubernetes
- cloud
---

####Pod là gì

"Pod" là một khái niệm ban đầu từ Google.

```
In Kubernetes, rather than individual application containers, Pods are the smallest deployable units that can be created, scheduled, and managed.

A Pod (as in a pod of whales or pea pod) corresponds to a colocated group of applications running with a shared context. Within that context, the applications may also have individual cgroup isolations applied. A Pod models an application-specific "logical host" in a containerized environment. It may contain one or more applications which are relatively tightly coupled -- in a pre-container world, they would have executed on the same physical or virtual host.
```
Ý tưởng chính đứng sau Pod là trong một kiến trúc microservice luôn luôn liên quan tới một vài chương trình "helper" như log, monitoring, cron, ... . Những chương trình giúp đỡ này được xây dựng để làm việc hợp tác với app. Vì vậy, thay vì chạy nhiều container độc lập , những tiến trình nên được chia sẻ chung namespace, mặc dù chúng được đóng gói trong các image khác nhau.

######Pod is the first class in Hyper

Trong hyper, một pod bao gồm một nhóm các docker image, được triển khai trong một HyperVM. Bên trong của instance, nhiều ứng dụng từ các image khác nhau chia sẻ các namespace: PID, network, IPC, UTS, User. Pod cung cấp một cái nhìn quen thuộc của một OS tới các ứng dụng, chứ không còn là triết lý "một process trên một container"

- Các process có thể thấy nhau

- Các process có thể sử dụng tất cả các phương tiện IPC để giao tiếp

- Các tiến trình chia sẻ cùng hostname

- Các tiến trình có truy cập tới tất cả các NIC gắn vào instance.

- Các tiến trình có truy cập vào tất cả các disk đã gắn vào instance

Ngoại trừ `Mount`. Bởi vì Pod có thể có nhiều app image, Hyper áp dụng việc cô lập `Mount` namespace


####Vòng đời của Pod
<img src="https://trello-attachments.s3.amazonaws.com/5562ba47387906ddef327e00/704x249/e4b1ec0168197c13e838bd5b405b3f28/pod.png">

Trong Hyper, một Pod có hai trạng thái:

- `Created`: Một Pod được định nghĩa, storage của nó được cấp phát, và các image Docker được download

- `Running`: Một Pod (với các container của nó) được khởi tạo trong một VM

Một pod có thể được khởi tạo một cách rõ ràng

```
hyper run -d podfile.json
```

Hoặc một cách khác

```
hyper run -t ubuntu
```

Trong cả hai cách, các Pods và VM không thể tách rời, Hyper sẽ tự động cung cấp một VM mới để làm chủ Pod, và Pod sẽ ở trạng thái `Running`

Tuy nhiên, cũng có thể tạo một Pod mà không tạo VM. Trong trường hợp này pod ở trạng thái `Created`

```
hyper create -p podfile.json
```

Có hai lựa chọn để start pod

```
hyper start pod_id

```

Lệnh `start` sẽ có một trigger để cung cấp một VM. Ngoài ra có thể chọn `replace` để chạy một pod mới

```
hyper replace -o old_pod_id -n new_pod_id
```

Trong trường hợp này, VM sẽ bị ngắt liên kết với `old_pod_id` và liên kết với `new_pod_id`. VM khi ấy đã running, `replace` là một lựa chọn nhanh hơn để khởi tạo một pod so với `run` và `start`

**Lưu ý: `old_pod_id` phải đang chạy**

Khi `stop` một Pod, VM bên dưới cũng sẽ bị chấm dứt

```
hyper stop pod_id
```

Khi đã stop, Pod sẽ trở về trạng thái `Created`

Để thực hiện destroy một Pod

```
hyper rm pod_id

```

Hyper sẽ (stop nếu cần thiết) xóa Pod và storage của nó.