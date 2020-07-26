---
layout: post
title: "[Write up ] Memory Forensic - CTF Nighst0rm 2017"
date: '2017-10-15 16:41:36'
---

Phải tới gần 1 năm rồi mình mới động vào CTF (truely). Đợt này nhóm NightSt0rm kết hợp với trường mình (KMA) tổ chức cuộc thi và Nova có đăng kí tham gia chơi với mục đích vui là chính!!!

Mình nhận định đề thi khá là khó ( mà lúc nào làm CTF mình thấy khó hết =)) không có cái nào dễ cả). Ngoại trừ 1 đội bá nhất ra thì các đội cũng same same nhau. 

Nhân tiện viết 1 bài mình đã làm được, vì bài này mình cũng mất khá nhiều thời gian và phải có hint từ BTC mới làm được. 

Đề bài ra với 1 file dump memory

Link: https://drive.google.com/file/d/0B7KprU81RDY7LXFtMVlCMV9QcEU/view?usp=sharing

Với 1 hint là Password được tìm thấy trong bài WeakAuth sẽ giúp cho bước cuối. 

-> Password tìm được trong bài WeakAuth là: `nightst0rm123`

Với memory forensic thì có 1 công cụ rất nổi tiếng đó là Volatility. 

Kiểm tra thông tin file dump.

```
volatility -f find-me.bin imageinfo
```

<img src="https://i.imgur.com/fUYdcSI.png">

Ta có đựoc profile của hệ thống đã được dump là `Win7SP0x86` hoặc `Win7SP1x86`. 

Một bước rất quan trọng đó là kiểm tra các process đang chạy

```
volatility -f find-me.bin --profile=Win7SP0x86 psscan
```


<img src="https://i.imgur.com/f8C63Qm.png">

Có 1 vài process nghi ngờ ở đây 

```
0x000000003f7ad030 iexplore.exe       2864   1336 0x3eb48420 2017-10-07 18:55:53 UTC+0000                                 
0x000000003fcd15d0 gpg-agent.exe      3576   3556 0x3eb48640 2017-10-07 18:45:41 UTC+0000                                 
0x000000003f664568 cmd.exe            1432   1336 0x3eb48200 2017-10-07 18:51:11 UTC+0000                                 
```

-> Cần kiểm tra lịch sử IE

-> Kiểm tra các command chạy trên CMD 

-> Khả năng có mã hóa gì đó nên chạy GPG Agent.


Kiểm tra lịch sử IE

```
volatility -f find-me.bin --profile=Win7SP0x86 iehistory
```

IE Mở tập tin chứa Flag ở dạng pdf 


<img src="https://i.imgur.com/mziCLJb.png">


Secret và Private Key 


<img src="https://i.imgur.com/5EwkoJ4.png">


Và đây là cái quan trọng, đường dẫn Drive

<img src="">

Đây chính là tập tin Pdf 

<img src="https://i.imgur.com/fKuhLvs.png">

Link: https://drive.google.com/file/d/0ByLZ4CxUZao1RVJBTWlVMDQtdEE/view

OK Fine. Kiểm tra thì File này ko phải pdf thường vì mở ra bị lỗi. 

Kiểm tra header thì ko có header gì, lệnh file cũng ko nhận diện ra được file loại nào. 

Dự đoán flag có thể ko phải file này, hoặc có thể đã bị mã hóa. 

Sâu chuỗi lại các thông tin ra có thể đoán được chính xác là file này bị mã hóa bằng hệ mật khóa công khai. 

Và private key thì có passphrase là password đã nói ở trên. 

List các file để tìm key nhưng không thấy trong list. 

```
volatility -f find-me.bin --profile=Win7SP0x86 filescan
```

Tìm mỏi mắt không biết nó ở đâu. Mình cũng dump process IE ra để kiểm tra xem nó còn bật ko nhưng cũng ko thấy. 

Sau khi có được hint của BTC nói rằng key được load lên RAM. 

Vi nó được bật bởi IE, do vậy private key và public key sẽ được load lên RAM tại thời điểm đó. 

Thực hiện `less` search với từ khóa `BEGIN` ( form của private và public key) 

Và đây là public key


<img src="https://i.imgur.com/2S5SKUv.png">

Tiếp tục search ta sẽ có Private key

<img src="https://i.imgur.com/GHvVuuS.png">

Import vào GPG


<img src="https://i.imgur.com/DVFTz3G.png">


Thực hiện decrypt file PDF

<img src="https://i.imgur.com/ppxeKYs.png">


Nhập passphrase `nightst0rm123`

Đây rồi 

<img src="https://i.imgur.com/KiDOBvG.png">

Decode base64 

<img src="https://i.imgur.com/vuNKbJU.png">

Flag: `NightSt0rm{M3m0ry_f0r3ns1cs_v3ry_3asy!}`


Ok. vì câu này mà mình đứng thứ 2 của cuộc thi. Thank hint :))