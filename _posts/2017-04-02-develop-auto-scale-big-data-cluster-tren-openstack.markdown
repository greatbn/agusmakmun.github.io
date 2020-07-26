---
layout: post
title: "[Tản mạn] Develop Auto Scale Big Data Cluster trên Openstack phần 1"
image: "/content/images/2017/05/OpenStack-Hadoop-Cloud-1.png"
date: '2017-04-02 06:13:33'
tags:
- openstack
- big-data
- openstack-sahara
- auto-scale
- hadoop
---

Đợt này mình nhận khá nhiều project liên quan tới dev, mặc dù mình không phải dân dev chuyên nhưng mình thích thử thách, do vậy mình cứ làm thôi. 
Vì không phải dân chuyên nên code khá ngu, hy vọng post lên đây sẽ có những cao nhân giúp mình tiến bộ hơn.
Như tiêu đề mình đề cập, thì trong Openstack có một project đảm nhiệm việc tạo, và quản lý các cluster Hadoop/Spark/... đó chỉnh là Project Sahara. 
Nhiệm vụ chính của sahara

 - Tạo/Xóa cluster
 - Scale Cluster
 - Excute Job trên Cluster

Ban đầu mình không muốn chọc vào code thằng này lắm, vì sợ không làm được (Nhìn đống source codegithub của nó cũng hiểu), một phần mình chưa vào giờ modify một project nào trong Openstack cả. 

Do vậy mình code theo hướng

 - Dựng bộ monitor các máy ảo
 - Viết tool check những instance nào đang vượt quá threshold
 - Kiểm tra instance đó thuộc cluster nào
 - Kiểm tra instance đó thuộc node group đang chạy process nào
 - Nếu có process đang chạy là datanode và không chạy đồng thời namenode thì scale, ngược lại không scale

Code này đã chạy và mình định không thay đổi gì. Nhưng sau khi xem xét thì bên kia họ lại yêu cầu hiển thị lên Horizon, và có thể set threshold theo từng cluster khác nhau. Khó nhằn rồi đây, nhưng khó cũng phải thử vậy. 

Vậy là đâm đầu đi vào đọc code thằng sahara, bản thân nó có 3 mã nguồn nhỏ là:

- [sahara](https://github.com/openstack/sahara)
- [shara-dashboard](https://github.com/openstack/sahara-dashboard)
- [python-saharaclient](https://github.com/openstack/python-saharaclient)

Vậy bắt đầu đọc từ đâu, câu hỏi thật mông lung. Nhưng mình đi theo yêu cầu của người ta trước, đó là hiển thị lên dashboard. 

##### Đọc code và make it run trong hơn 1 ngày

Bắt đầu lần mò vào dashboard thấy các tab của sahara hiển thị đều chia thành các folder khác nhau


<img src="http://i.imgur.com/Aom7g5X.png)">

Hay rồi, khả năng cao sẽ mò được code trong này, tiếp tục tìm trong file `clusters/workflow/create.py`

File này sẽ thực hiện implement chức năng `Launch Cluster`

Bắt đầu sửa thêm các fields cần thiết

<img src="http://i.imgur.com/aSP6Mls.png)">

Hàm này sau khi lấy thông tin nó sẽ call tới function `cluster_create` trong module `api/sahara`

<img src="http://i.imgur.com/BRhI7mo.png)">

Do vậy ta cũng sẽ sửa luôn function `api/sahara/cluster_create`

<img src="http://i.imgur.com/iq0RlSl.png">

Sửa phần hiển thị details của cluster trong `data_processing/clusters/templates/clusters/_details.html`


<img src="http://i.imgur.com/cHtzDD5.png">

Ok, vậy là ta có thể tạo cluster từ dashboard, với các tham số truyền vào như sau 

<img src="http://i.imgur.com/8gsyp2h.png)">

Và có thể hiển thị chi tiết trong `General Info`

<img src="http://i.imgur.com/bdSGyyT.png)">

Nhưng khoan, chưa tạo cluster đâu nhé, cần sửa tiếp mã nguồn của `sahara` và `python-saharaclient` mình sẽ viết ở phần sau.

Cuối cùng mình nhận thấy việc sửa code của openstack cũng không phải khó, cũng không hẳn là dễ, nhưng nếu chịu khó thì vẫn làm được. 

Phần 2: https://sapham.net/tan-man-develop-auto-scale-big-data-cluster-tren-openstack-phan-2/

