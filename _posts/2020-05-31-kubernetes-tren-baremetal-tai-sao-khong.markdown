---
layout: post
title: Kubernetes trên Baremetal tại sao không?
date: '2020-05-31 14:53:41'
---

Hẳn là chủ đề này cũng không còn hot nữa. Nhưng mới đây mình mới triển khai hệ thống Kubernetes trên Baremetal và yêu cầu cũng đầy đủ như việc Kubernetes trên các nền tảng cloud. Dĩ nhiên là không thể đầy đủ 100% được. 

Khi dùng trên Kubernetes của 1 nhà cung cấp Cloud nào đó (AWS, GCP, Azure, DO, ...) hoặc Openstack thì thường các nhà cung cung cấp và cộng đồng sẽ cố gắng việc tích hợp sâu các thành phần cần thiết cho Kubernetes. Ví dụ như Service Type Load Balancer, Persistent Volume. Mọi thứ hoàn toàn tự động và thật sự hoàn hảo. Cost cho việc quản trị 1 hoặc 1 vài Cluster kubernetes giảm rất nhiều. 

Đa phần các nhà cung cấp đều có dịch vụ Managed Kubernetes, nên Engineer cũng lười hơn rất nhiều. Hẳn là giờ chỉ chỉnh sửa Yaml và kubectl apply. Nhưng việc triển khai Kubernetes trên Baremetal cũng khá vất vả. 

- Không thẻ tự động tạo Load Balancer khi tạo Service Type là Load Balancer
- Không tự động tạo Persistent Volume khi tạo PV hay PVC.
- Không tự động attach Volume vào đúng host.
- Lựa chọn provisioning tool cũng là việc cần cân nhắc cho khả năng mở rộng và thay đổi sau này.

Dựa trên kinh nghiệm mình, mình chia sẻ với stack mình đang build như sau

- Đối với Load Balancer thì sử dụng cặp L4LB - Haproxy hoặc Nginx là đủ. Cặp LB này sẽ đứng bên ngoài nhận mọi request từ bên ngoài và forward xuống *-Ingress (Nginx, Traefik, ..)

- Đối với Volume thì sử dụng NFS (đơn giản nhất), nếu có hạ tầng storage và muốn dựng Software-Defined Storage thì mình lựa chọn CEPH và sử dụng CEPH-CSI [1].

Vậy đối với các nhu cầu sử dụng SAN thì sao. Thường thì các hãng sẽ có viết các CSI (Container Storage Interface) để hỗ trợ việc tích hợp vào K8s một cách dễ dàng. Các bạn có thể tham khảo các CSI hiện tại đang hỗ trợ tại [2].

Nhưng nếu SAN bạn không được Vendor hỗ trợ thì sao? Đây là case mình gặp, sau khi tìm hiểu thì mình thấy có Ember-CSI [3]. Đây là 1 Driver được viết và tận dụng các driver của Openstack Cinder. Như các bạn đã biết thì Cinder hỗ trợ rất nhiều hệ thống Storage khác nhau [4]. Do vậy việc bạn chỉ cần kiểm tra Cinder có hỗ trợ Storage Backend đó hay không và kết hợp dùng Ember-CSI là xong. 

- Đối với Provisioning tool, hiện tại thì Kubeadm đã hỗ trợ vệc deploy High Available cho Kube-Master nên mình sẽ lựa chọn kubeadm. Nhưng trước khi kubeadm hỗ trợ vậy thì mình sẽ chọn RKE [5], đây là một tool mà theo đánh giá của nhiều anh em rất thích. Do tính custom và quản lý theo dạng IaC nên việc manage rất ngon.

Cùng thảo luận về vấn đề này nào?  Vì theo mình đánh giá việc chạy Kubernetes trên Cloud thì cũng không có nhiều vấn đề lắm :)) hy vọng không động chạm ai.


### Tham khảo

[1] - https://github.com/ceph/ceph-csi

[2] - https://kubernetes-csi.github.io/docs/drivers.html

[3] - https://ember-csi.io/

[4] - https://docs.openstack.org/cinder/ussuri/drivers.html

[5] - https://github.com/rancher/rke