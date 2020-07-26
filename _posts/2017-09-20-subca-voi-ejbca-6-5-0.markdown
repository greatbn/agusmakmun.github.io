---
layout: post
title: SubCA với EJBCA  6.5.0 - Phần 2
image: "/content/images/2017/09/Screenshot-from-2017-09-21-00-19-16.png"
date: '2017-09-20 17:19:54'
tags:
- ejbca
- pki
- certificate
- rootca
- subca
- cau-hinh-subca
---

Hi all, như bài trước mình đã đề cập tới việc xây dựng hệ thống PKI bao gồm 2 subCA và 1 rootCA. Nếu bạn nào chưa biết [cài đặt EJBCA có thể theo dõi post](https://sapham.net/cai-dat-ejbca-6-5-0-5/) trước của mình.

Phần này mình sẽ nói nhiều hơn về một số concept trong EJBCA, nếu ai chưa hiểu có thể vào documents bên dứoi:

- [EJBCA Concepts and Terminology](https://www.ejbca.org/docs/index.html#Concepts and Terminology)

Mô hình hệ thống sẽ như sau

<img src="https://i.imgur.com/BEb89PE.png">

####Lý thuyết triển khai

Mình sẽ nói qua một chút về lý thuyết cũng như các bước sẽ thực hiện để triển khai hệ thống này 

- Các CA (bao gồm rootCA và subCA) đều cài đặt EJBCA như hướng dẫn trên 

- Trên RootCA thực hiện tạo Cert Self-Sign, rootCA tự ký nó. 

- Trên SubCA thực hiện import Cert của rootCA

- SubCA tạo subCA với issuer là rootCA bên trên, profile là SUBCA (có sẵn)

- subCA tạo certificate request, trên rootCA tạo 2 entity tuơng ứng với 2 subCA để subCA thực hiện request Cert qua Public Website.

- subCA import cert nhận được từ rootCA và hoàn thành quá trình cài đặt

- Tạo ceritificate profile, entity profile cho subCA, tạo entity và request cert kiểm tra chứng thực.

######Tạo ROOTCA

Truy cập vào trang Admin của rootCA, nằm tại đường dẫn `https://<hostname>:8443/ejbca/adminweb/`

Xác thực bằng superadmin key của rootCA.

Chọn Ceritifcate Authorities, nhập tên CA tuơng ứng vào ô `Add CA` và chọn Create 

<img src="https://i.imgur.com/qPYT1YX.png">

Sau khi nhấn Create, ta sẽ thực hiện cấu hình các thông số như sau

<img src="https://i.imgur.com/HxHbUXi.png">

Chọn tiếp Create và hoàn thành cài đặt RootCA

<img src="https://i.imgur.com/PPqJ4qc.png">

Lúc này ta có thể vào Public Web tại địa chỉ `http://<hostname>:8080/ejbca/` và lấy Certificate của rootCA về để import vào các subCA1 và subCA2.

Mục `Retrieve` -> `Fetch CA certificates` chọn `Download PEM chain`

<img src="https://i.imgur.com/ftAFRb3.png)">

Ok. tới lúc này ta đã có certificate chain của RootCA. 

#####Tạo entity cho 2 SubCA

Ta sẽ thực hiện tạo 2 entity cho 2 subCA tuơng ứng

- CN=SAPHAM.SUBCA1 - Profile: SUBCA
- CN=SAPHAM.SUBCA2 - Profile: SUBCA

Trở lại trang Admin, thực hiện tạo profile cho các entity này, 

Mở `End Entity Profiles`, nhập SUBCA vào ô `Add Profile` và chọn Add

<img src="https://i.imgur.com/WdHKl35.png">

Chọn Profile này và Edit End Entity Profile 

Cấu hình như sau và chọn Save

<img src="https://i.imgur.com/VHPHBgj.png">

Tiếp tục chọn `Add End Entity`

Chọn Profile vừa tạo và Nhập thông tin tuơng ứng với 2 SUBCA như sau

<img src="https://i.imgur.com/iwY9dne.png">

Chọn Add để lưu các thông tin. 

##### SubCA

Tiếp theo truy cập vào trang Admin của các subCA và import cert của rootCA.

Tại trang `Certification Authorities` chọn `Import CA certificate`

<img src="https://i.imgur.com/w5lYI1O.png)">

Kết quả như sau 

<img src="https://i.imgur.com/jr59dRM.png">

Tiếp tục tạo SUBCA tuơng ứng 

Nhập thông tin như sau

<img src="https://i.imgur.com/7i0fP3u.png">

Tạo certificate request, ở bước này ta cần import CA chain của ROOTCA lần nữa và chọn `Make Cerficate Request`

<img src="https://i.imgur.com/Mv8dc3g.png)">

Có thể lưu ở dạng file hoặc copy request content này. 

Truy cập vào Public WEB của rootCA, chọn mục `Create Certificate from CSR`

Nhập thông tin như sau

<img src="https://i.imgur.com/OM1z9MO.png">

Lưu ý: Result type = PEM - certificate only

Và chọn OK. 

<img src="https://i.imgur.com/h04jMBu.png">

Lưu certificate file của subCA. 

Trở lại trang Admin của subCA, lúc này trạng thái subCA đang như sau:

<img src="https://i.imgur.com/E654WwU.png)">

Ta cần import cert đã được rootCA ký vào subCA này, chọn `Edit CA` SUBCA1.

<img src="https://i.imgur.com/NyfhkqM.png">


Thực hiện `Bước 2`, chọn cert file vừa xin từ rootCA và chọn `Receive Cerificate Response`

Và kết quả như sau là thành công

<img src="https://i.imgur.com/XVaSfwN.png">

Kiểm tra struct của CA tại trang `CA Structure and CRLs`

<img src="https://i.imgur.com/2MnoaPX.png">

SAPHAM.SUBCA2  sẽ thực hiện tương tự. 

Lúc này ta có thể kiểm tra certificate được cấp bở subCA1 và phần certificate path sẽ như sau

<img src="https://i.imgur.com/LPYPuy1.png)">

Ở bài sau, mình sẽ viết về việc cấu hình LDAP để lưu giữ các thông tin của Entity và việc sử dụng certificate để kí và mã hóa dữ liệu như thế nào...   ;)