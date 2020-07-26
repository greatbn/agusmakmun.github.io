---
layout: post
title: Đánh index cho LDAP để search nhanh hơn
date: '2018-04-07 02:26:23'
---

LDAP Server là một máy chủ chạy dịch vụ tương tự như một cuốn danh bạ điện thoại. LDAP được sử dụng khá rộng rãi ( ngoài ra AD của M$ cũng là một Directory Server rất mạnh mẽ).

Do đa số việc cài và sử dụng ban đầu thì không có gì khó khăn nhưng sau một thời gian, số lượng bản ghi nhiều lên làm cho việc tìm kiếm và xác thực user bị chậm đi ta cần phải cấu hình một vài tham số. 

Khi rà soát LDAP tôi có phát hiện ra các dòng log như sau tại syslog

```
<= bdb_equality_candidates: (member) not indexed
<= bdb_equality_candidates: (sn) not indexed
<= bdb_equality_candidates: (cn) not indexed
```

Ở đây có 3 fields không được đánh index là `cn`, `sn` và `member`. 

làm cho việc xác thực bị chậm đi khá nhiều..

Ta có thể kiểm tra các field đã được đánh index bằng lệnh sau 

```
~# /usr/sbin/slapcat -n 0 | grep olcDbIndex
olcDbIndex: objectClass eq
olcDbIndex: entryUUID  eq
olcDbIndex: entryCSN  eq
```

Thực hiện đánh index cho các field bị thiếu trên

Tạo file ldif như sau

```
dn: olcDatabase={1}hdb,cn=config
changetype: modify
add: olcDbIndex
olcDbIndex: cn eq
olcDbIndex: member eq
olcDbIndex: sn eq
```

Add index

```
ldapadd -Y EXTERNAL -H ldapi:/// -f add_index.ldif
```

Kiểm tra lại các trường đã được index

```
~# /usr/sbin/slapcat -n 0 | grep olcDbIndex
olcDbIndex: objectClass eq
olcDbIndex: entryUUID  eq
olcDbIndex: entryCSN  eq
olcDbIndex: cn eq
olcDbIndex: member eq
olcDbIndex: sn eq
```

Lúc này ta có thể kiểm tra việc xác thực tới LDAP nhanh hơn rất nhiều

Before

<img src="https://i.imgur.com/CxIdbXi.png">

After

<img src="https://i.imgur.com/CR1PIcM.png">