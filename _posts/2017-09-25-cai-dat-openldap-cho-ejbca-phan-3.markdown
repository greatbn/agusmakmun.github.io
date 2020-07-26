---
layout: post
title: Cài đặt OpenLDAP -  EJBCA Phần 3
date: '2017-09-25 06:10:58'
tags:
- ejbca
- subca
- rootca
- pki
---

####Cài đặt OpenLDAP

- Chuẩn bị

  - Ubuntu Server 14.04

- Update Repo

`sudo apt-get update && apt-get upgrade -y`

- Cài đặt OpenLDAP

`sudo apt-get install -y slapd ldap-utils`

- ở bước cài đặt này OpenLDAP Sẽ hỏi password admin của LDAP Server. Ta có thể bỏ qua hoặc nhập vào

- Sau khi cài đặt xong ta cấu hình lại LDAP

`sudo dpkg-reconfigure slapd`

- Chọn No

<img src="https://i.imgur.com/uavxrKS.png">

- Nhập vào domain

<img src="https://i.imgur.com/8dQLD8f.png)">

- Nhập vào Tên tổ chức

<img src="https://i.imgur.com/R1wBcj5.png">

- Nhập admin password

<img src="https://i.imgur.com/cnLVnv8.png">

- Nhập admin password confirm

<img src="https://i.imgur.com/EiQglsT.png)">

-  Chọn HDB

<img src="https://i.imgur.com/uRrwwYe.png">

- Chọn Yes

<img src="https://i.imgur.com/tH8igM1.png)">

- Chọn Yes

<img src="https://i.imgur.com/05XzGaa.png)">

- Chọn No

<img src="https://i.imgur.com/jDBSPGO.png">


- Sau khi xong ta có thể dùng lệnh `slapcat` để kiểm tra lại các thông tin

<img src="https://i.imgur.com/D6zOk8K.png)">

- Tạo OU People chứa các user và OU Groups chứa các nhóm

- Sử dụng file ldif có nội dung như sau

```
dn: ou=People,dc=vnpt,dc=vn
objectClass: organizationalUnit
ou: People

dn: ou=Groups,dc=vnpt,dc=vn
objectClass: organizationalUnit
ou: Groups

```

- Lưu với tên base.ldif

- Add vào LDAP Server

`ldapadd -x -D cn=admin,dc=vnpt,dc=vn -W -f base.ldif`

- Nhập password Admin

<img src="https://i.imgur.com/9w8Wwrw.png)">


####Cấu hình chặn anon user query LDAP Server

- Tạo một file ldif có nội dung như sau

`vi /usr/share/slapd/ldap_disable_bind_anon.ldif`

```
dn: cn=config
changetype: modify
add: olcDisallows
olcDisallows: bind_anon

dn: cn=config
changetype: modify
add: olcRequires
olcRequires: authc

dn: olcDatabase={-1}frontend,cn=config
changetype: modify
add: olcRequires
olcRequires: authc

```

- Add vào LDAP Server

`ldapadd -Y EXTERNAL -H ldapi:/// -f /usr/share/slapd/ldap_disable_bind_anon.ldif`


#### End

Bài này mình viết cách đây đã hơn 1 năm, lúc mình còn ở VNPT do vậy example liên quan tới VNPT.  Chủ yếu để theo mạch của series.

Việc cấu hình LDAPA cho subCA server cũng tuơng tự như trên. Chỉ khác base DN đối với mỗi subCA.

Ví dụ
- DN cho subCA 1

`dc=subca1, dc=sapham, dc=net`

- DN cho subCA 2

`dc=subca2, dc=sapham, dc=net`