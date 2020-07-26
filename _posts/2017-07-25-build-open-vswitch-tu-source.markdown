---
layout: post
title: Build Open vSwitch từ source code
date: '2017-07-25 02:52:01'
---

#### Build Open VSwitch từ source code.

Vì một vài lý do, mình cần sử dụng OpenvSwitch phiên bản mới nhất. Do vậy đây chỉ là những dòng note lại các bước build OpenVSwitch từ source code..


**1. Cài đặt các dependencies**

```sh
apt install gcc autoconf automake libtool make -y
```

**3. Clone mã nguồn từ Git**

```sh
git clone https://github.com/openvswitch/ovs
```

**2. Bootstrap Openvswitch**

```sh
cd ovs
./boot.sh
```

**2. Configure**


```sh
./configure --prefix=/usr --localstatedir=/var --sysconfdir=/etc
```

**3. Install**

```
make
make install
```

**4. Enable module openvswitch**

```
modprobe openvswitch
```

**5. Start OvS**


```
/usr/share/openvswitch/scripts/ovs-ctl start
```

**6. Let's Check**

<img src="http://i.imgur.com/xu2Zi5s.png">