---
layout: post
title: Các tip hữu ích khi sử dụng ansible
date: '2017-05-18 16:50:50'
tags:
- ansible
- config-management
---

Dưới đây là một số tip mình dùng khi sử dụng ansible.

##### VIM with YAML

Nếu bạn đã từng viết yaml trên VIM sẽ thấy một điều rất bực là tabsize default của VIM là 8 space. Mà syntax của yaml tabsize chỉ là 2space, nên nếu như không tinh chỉnh vim thì ta sẽ viết rất mệt. Ta chỉ cần thực hiện set tabsize thành 2 space khi file type là yaml như sau

Mở `~/.vimrc` thêm vào dòng sau

```
autocmd FileType yaml setlocal ai ts=2 sw=2 et
```
Sau đó mở yaml lên và thưởng thức

##### Check syntax

`ansible-playbook` hỗ trợ cho ta việc check syntax của toàn bộ 1 playbook

```
ansible-playbook --syntax-check <path-to-yaml-playbook>
```

Nếu không có lỗi gì sẽ trả về tên của tập tin playbook đó

##### Step By Step 

Ta cũng có thể chạy một playbook step by step, tới mỗi bước, ansible-playbook sẽ cho ta 3 lựa chọn: `y`, `n`, `c`

  - y = Yes: Thực hiện task này
  - n = No: Không thực hiện task này 
  - c = Continue: Chạy toàn bộ playbook mà không hỏi

```
PLAY [controller] **************************************************************
Perform task: TASK: setup (N)o/(y)es/(c)ontinue: N

Perform task: TASK: setup (N)o/(y)es/(c)ontinue: *******************************
Perform task: TASK: patch-cinder-ocata : Apply patch for cinder-volume Ocata (N)o/(y)es/(c)ontinue: y

Perform task: TASK: patch-cinder-ocata : Apply patch for cinder-volume Ocata (N)o/(y)es/(c)ontinue: ***

TASK [patch-cinder-ocata : Apply patch for cinder-volume Ocata] ****************
ok: [capt-controller-1]
ok: [capt-controller-3]
ok: [capt-controller-2]
Perform task: TASK: patch-cinder-ocata : file (N)o/(y)es/(c)ontinue: c

Perform task: TASK: patch-cinder-ocata : file (N)o/(y)es/(c)ontinue: ***********

```

