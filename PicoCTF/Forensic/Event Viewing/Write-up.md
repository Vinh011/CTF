# Write up Event Viewing - picoCTF
**Loại:** Phân tích nhật ký (Log Analysis)

## 1. Mô tả 
- Họ cài một phần mềm tải từ internet
- Chạy phần mềm nhưng không thấy gì xảy ra
- Sau đó mỗi lần login máy tự shutdown ngay

<img width="900" height="734" alt="image" src="https://github.com/user-attachments/assets/232f5aea-67f0-4cf1-a6ca-50a57f5e68cf" />

---

Đề bài cho 1 file log của Window

---

## 2. Bước giải 

### Cách 1

Mở file bằng **Event Vỉewer** xuất hiện giao diện như trong ảnh, rồi tôi lưu file lại với định dạng **.csv** để mở trong trình đọc

<img width="950" height="684" alt="image" src="https://github.com/user-attachments/assets/57fe767b-a067-4079-acd4-6d16bdf490ee" />


---

Theo như mô tả của đề bài, tôi tìm kiếm chuỗi **shutdown** và tìm được base64.

```
Information,16/07/2024 12:02:35 SA,User32,1074,None,"The process C:\Windows\system32\shutdown.exe (DESKTOP-EKVR84B) has initiated the shutdown of computer DESKTOP-EKVR84B on behalf of user DESKTOP-EKVR84B\user for the following reason: No title for this reason could be found
 Reason Code: 0x800000ff
 Shutdown Type: shutdown
 Comment: dDAwbF84MWJhM2ZlOX0="
```

Decode:

```
t00l_81ba3fe9}
```

---

Có lẽ đây là 1 đoạn **flag** như mô tả đề bài, sau khi biết **flag** được mã hóa bằng base64, với định dạng flag là **picoCTF{** tôi encode nó thành base64 để tìm kiếm 

```
Information,15/07/2024 10:55:57 CH,MsiInstaller,1033,None,Windows Installer installed the product. Product Name: Totally_Legit_Software. Product Version: 1.3.3.7. Product Language: 0. Manufacturer: cGljb0NURntFdjNudF92aTN3djNyXw==. Installation success or error status: 0.
```

Decode:

```
picoCTF{Ev3nt_vi3wv3r_
```


Tôi đã thử ghép các đoạn flag này nhưng không đúng, có vẻ còn thiếu !

---

Tìm tiếp được đoạn:

```
	Object Name:		\REGISTRY\MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
	Object Value Name:	Immediate Shutdown (MXNfYV9wcjN0dHlfdXMzZnVsXw==)
	Handle ID:		0x208
	Operation Type:		New registry value created
```

Decode:

```
1s_a_pr3tty_us3ful_
```

---

Flag: **picoCTF{Ev3nt_vi3wv3r_1s_a_pr3tty_us3ful_t00l_81ba3fe9}**

## Cách 2

Tôi dùng tool để chuyển file sang định dạng **.xml**

```bash
evtx_dump.py Windows_Logs.evtx > logs.xml
```

Sau khi chạy sẽ có file:

```
logs.xml
```

---

Tìm event cài phần mềm: **EventID 1033**

```bash
grep -A30 1033 logs.xml
```

Output sẽ có chuỗi:

```
cGljb0NURntFdjNudF92aTN3djNyXw==
```

Decode base64:

```bash
echo cGljb0NURntFdjNudF92aTN3djNyXw== | base64 -d
```

Kết quả:

```
picoCTF{Ev3nt_vi3wv3r_
```

Đây là đoạn flag 1

---

Tìm resistry bị sửa.
- Hacker tạo persistence bằng Registry vì nó giúp malware tự chạy lại mỗi khi máy khởi động hoặc khi user đăng nhập

**EventID: 4657**

```bash
grep -A30 4657 logs.xml
```

Sẽ thấy: 

```
MXNfYV9wcjN0dHlfdXMzZnVsXw==
```

Decode:

```bash
echo MXNfYV9wcjN0dHlfdXMzZnVsXw== | base64 -d
```

Kết quả: 

```
1s_a_pr3tty_us3ful_
```

Đây là đoạn flag thứ 2

---

Tìm shudown
- Máy bị shutdown khi login.

**EvenID: 1074**

Search:

```
grep -A20 1074 logs.xml
```

Tìm được base64:

```
dDAwbF84MWJhM2ZlOX0=
```

Decode:

```bash
echo dDAwbF84MWJhM2ZlOX0= | base64 -d
```

Kết quả: 

```
t00l_81ba3fe9}
```

Ghép flag: **picoCTF{Ev3nt_vi3wv3r_1s_a_pr3tty_us3ful_t00l_81ba3fe9}**








