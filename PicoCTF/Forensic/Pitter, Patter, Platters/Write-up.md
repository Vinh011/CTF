# picoCTF - Pitter, Patter, Platters (Write-up)

## Thông tin bài
- Category: Forensics
- Difficulty: Medium
- File: [`suspicious.dd.sda1`](https://challenge-files.picoctf.net/c_shape_facility/694515a76dfa29953964179c00823d28ba592dbddeac76df3b4ae6fd9414f6dc/suspicious.dd.sda1)

---

## Mục tiêu
Tìm flag bị giấu trong disk image.

---

## Hint phân tích
It may help to analyze this image in multiple ways:
- As a blob
- As an actual mounted disk  
Have you heard of slack space?

Insight:
- Không chỉ xem file bình thường
- Phải phân tích filesystem + raw data
- Flag nằm trong slack space

---

# Kiến thức cần biết

## Slack Space là gì?
- File nhỏ hơn block size
- Phần còn dư trong block = slack space
- Có thể chứa dữ liệu cũ hoặc bị giấu

---

# Cách 1: Autopsy (GUI)

## Load image
- New Case → Add Data Source
- Chọn file suspicious.dd.sda1

## Tìm file
Views → File System
→ suspicious-file.txt

<img width="1919" height="1017" alt="image" src="https://github.com/user-attachments/assets/9f3a0241-46ce-4753-973c-5db7dfc6cbb7" />


Nội dung:
Nothing to see here! But you may want to look here -->

## Bật Slack

<img width="1917" height="1018" alt="image" src="https://github.com/user-attachments/assets/3cf72c98-6c4a-48a3-969a-121ed31a1605" />


## Tìm slack file
suspicious-file.txt-slack

<img width="1919" height="1018" alt="image" src="https://github.com/user-attachments/assets/7816bba6-ff40-4ece-9061-14c64c96b778" />

## Nội dung:
```
0x00000000: 7D 00 33 00  39 00 38 00   36 00 33 00  31 00 32 00    }.3.9.8.6.3.1.2.
0x00000010: 66 00 5F 00  33 00 3C 00   5F 00 7C 00  4C 00 6D 00    f._.3.<._.|.L.m.
0x00000020: 5F 00 31 00  31 00 31 00   74 00 35 00  5F 00 33 00    _.1.1.1.t.5._.3.
0x00000030: 62 00 7B 00  46 00 54 00   43 00 6F 00  63 00 69 00    b.{.F.T.C.o.c.i.
0x00000040: 70 00 00 00  00 00 00 00   00 00 00 00  00 00 00 00    p...............
```

---

# Cách 2: CLI (Sleuth Kit)

## Liệt kê file
```bash
fls suspicious.dd.sda1
```

Kết quả:

```
└─$ fls suspicious.dd.sda1
d/d 11: lost+found
d/d 2009:       boot
d/d 4017:       tce
r/r 12: suspicious-file.txt
V/V 8033:       $OrphanFiles

```

## Đọc file

```bash
icat suspicious.dd.sda1 12
```

## Tìm offset
```bash
strings -a -t x suspicious.dd.sda1 | grep "Nothing"
```

Offset là: **200400**


## Dump dữ liệu
```bash
xxd -s 0x200400 -l 200 suspicious.dd.sda1 
```

Kết quả: 

```
00200400: 4e6f 7468 696e 6720 746f 2073 6565 2068  Nothing to see h
00200410: 6572 6521 2042 7574 2079 6f75 206d 6179  ere! But you may
00200420: 2077 616e 7420 746f 206c 6f6f 6b20 6865   want to look he
00200430: 7265 202d 2d3e 0a7d 0033 0039 0038 0036  re -->.}.3.9.8.6
00200440: 0033 0031 0032 0066 005f 0033 003c 005f  .3.1.2.f._.3.<._
00200450: 007c 004c 006d 005f 0031 0031 0031 0074  .|.L.m._.1.1.1.t
00200460: 0035 005f 0033 0062 007b 0046 0054 0043  .5._.3.b.{.F.T.C
00200470: 006f 0063 0069 0070 0000 0000 0000 0000  .o.c.i.p........
```

> Flag bị đảo ngược
---


# Flag

picoCTF{slack_space_is_fun_4f3b0b1e}

---
