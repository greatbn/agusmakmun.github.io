---
layout: post
title: Sử dụng byobu để quẩy trên terminal dễ dàng hơn
image: "/content/images/2016/08/byobu.png"
date: '2016-08-06 06:14:15'
tags:
- linux
- byobu
- termial
---

Hẳn là nhiều anh em làm sysadmin hay không phải sysadmin nhưng hay sử dụng termial đều mong muốn termial có nhiều chức năng ví dụ như cắt đôi màn hình, mở tab nhanh, bla bla... Nhưng chỉ một số (rất ít) distro quan tâm tới việc làm sao để terminal nhiều chức năng nhất.

Cách đây vài tháng mình biết tới byobu. Đó cũng là lúc mình không dùng [deepinOS](deepin.org) nữa và do đó byobu là một công cụ hỗ trợ rất nhiều cho mình.

Chưa cần biết nó là gì? Cứ cài đặt rồi trả nghiệm.

Trên Ubuntu 

```
apt-get install -y byobu
```

Trên CentOS 

```
yum install -y byobu
```

Để mở ta gõ `byobu` và đây là giao diện. 

<img src="http://i.imgur.com/wFOU9wp.png)">

Nhìn thì chả có gì khác nhỉ. Ta thấy có một dòng dưới cùng có các thông tin về các tab đang mở, ngày giờ, CPU, RAM...

Ta cần cái gì đó để biết cách sử dụng nó như nào.. Hãy nhấn `Shift + F1` để mở Help

<img src="http://i.imgur.com/gzo9HdV.png)">

Ở đây ta sẽ thấy hầu hết các chức năng của byobu và những phím tắt để thực hiện nó. Ban đầu ta chưa cần biết hết. Nhưng mình sẽ liệt kê một số phím mình hay dùng, rồi các bạn sẽ tự khám phá ra các chức năng tiếp theo..

Ok. Ta quay trở lại màn hình ban đầu nhấn `q`

- Nhóm Phím `F2`

Để mở một tab mới ta nhấn `F2`. Vậy là ta đã có 2 tab `0` và `1`

<img src="http://i.imgur.com/ny7MA8z.png)">

Để Chia đôi màn hình theo chiều dọc ta nhấn `Ctrl + F2`

<img src="http://i.imgur.com/ho1AB7b.png)">

Để chia đôi màn hình theo chiều ngang ta nhấn `Shift + F2`

<img src="http://i.imgur.com/aFyRYks.png)">

- Nhóm phím `F3/F4`

Ở trên ta đã mở 2 tab. Làm thế nào để di chuyển nó. Ta sẽ dùng cặp phím `F3/F4` để di chuyển.

Nhưng mình hay dùng cặp phím `Alt + Right Arrow` và `Alt + Left Arrow` để di chuyển vì nó gần hơn  :D 

Để di chuyển giữa các cửa sổ mà ta đã chia đôi.
Tôi thường hay sử dụng cặp phím `Shift + Up Arrow` và `Shift + Down Arrow`

- Phím `F6`

Làm sao để đóng một cửa sổ hay một tab.  Tôi thường hay sử dụng `Ctrl + F6` để đóng một cửa sổ hay một tab đang dùng.

- Phím `F7`

Bạn đang `tail` một đoạn log và muốn di chuyển lên để check. Không cần dùng chuột nữa. Chỉ cần nhấn `F7` Rồi dùng các phím `Arrow` Và `Page UP` + `Page Down` để xem.

- Phím `F8`

Khi ta mở rất nhiều tab. Làm sao để nhớ được tab này. Byobu cho phép ta đặt tên cho tab đó. Để đặt tên cho tab đang mở ta chỉ cần nhấn `F8` và gõ tên cho tab đó. 

<img src="http://i.imgur.com/sphIDuI.png)">

#####Trick

Bạn phải SSH nhiều vào Server. Bạn đôi khi cần phải tạo nhiều session trên server đó. Bạn đi làm, nhưng đang dở việc thì phải về nhà nhưng tắt session SSH đi thì coi như hết. Đừng lo đã có Byobu. Bạn hãy cài Byobu lên server đó. Rồi bật lên. Ta có thể quẩy trên server mà không cần ssh nhiều lần, Không cần lo công việc dang dở mà phải về nhà. Vì byobu vẫn chạy trên đó. Ta đóng máy về nhà mở lên lại như vậy..

#####Kết luận

Byobu là một tool không thể thiếu trên terminal. Giúp ta làm việc hiệu quả hơn rất nhiều.

