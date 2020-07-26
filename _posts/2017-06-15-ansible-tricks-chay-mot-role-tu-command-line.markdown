---
layout: post
title: Ansible tricks - Chạy một role từ command line
image: "/content/images/2017/06/events_scm_ansible_logo.png"
date: '2017-06-15 17:25:18'
tags:
- ansible
- tricks
- devops
---

Khi viết một role cho ansible role, thi thoảng tôi muốn chạy luôn role này. Hoặc có 1 role tôi clone được từ github. Đôi khi chỉ có một role thôi nên rất ngại viết hẳn một playbook đẻ chỉ chạy mỗi role đó. 
`ansible-role` sẽ cứu rỗi ta, làm công việc nhanh chóng hơn.

Cài đặt `ansible-role`


```
sudo pip install ansible-role
```

Sau khi cài đặt, ta thấy `ansible-role` có các options như 

<img src=http://i.imgur.com/DoeXTDi.png>

#### Ví dụ 

- Ta có 1 role `ntp-client`

<img src=http://i.imgur.com/KResLoC.png>

Muốn chạy role này ta thực hiện lệnh sau 

```
ansible-role --module-path .  ntp-client  ntp -i hosts -u root
```

- `--module-path`: Đường dẫn tới module
- `ntp-client`: tên của role
- `-i hosts`: inventory host
- `-u root`: remote user 

Kết quả 


<img src="http://i.imgur.com/HMoa1kJ.png">

Ta có thể thấy hiện tại `ansible-role` đã hỗ trợ tất cả các tham số của `ansible-playbook`, do vậy ta có thể thoải mái làm việc với `ansible-role` mà ít bị hạn chế bởi những options. 