# CTF Write-up: Name Calling
<img width="625" height="502" alt="image" src="https://github.com/user-attachments/assets/9e9c47b5-6769-4034-8407-1b1af1a21f4a" />

---

## Mô tả challenge

```
I think someone called you chicken. You should do something about it.
File: yousaidwhat.pcapng
```

Challenge cho một file `.pcapng` — đây là định dạng capture gói tin mạng (packet capture). Nhiệm vụ: phân tích lưu lượng mạng để tìm flag.

---

## Giải

---

### Xem tổng quan giao thức

**Lệnh:**
```bash
tshark -r yousaidwhat.pcapng -q -z io,phs
```

**Giải thích từng phần:**
- `tshark` — Wireshark command-line, đọc và phân tích pcap
- `-r yousaidwhat.pcapng` — **r**ead file này
- `-q` — **q**uiet mode, không in từng packet ra màn hình
- `-z io,phs` — **z**=statistics, `io,phs` = **Protocol Hierarchy Statistics** (thống kê cây giao thức)

**Kết quả:**
```
eth                        frames:373 bytes:290354
  ip                       frames:334 bytes:287308
    udp                    frames:37
      ssdp                 frames:5
      mdns                 frames:10
      ntp                  frames:2
    icmp                   frames:6
    tcp                    frames:291
      http                 frames:28 bytes:60761
        data-text-lines    frames:10
        media              frames:1
        image-jfif         frames:3
  arp                      frames:28
```

**Phân tích:**
- 373 gói tin tổng cộng, chủ yếu là IP/TCP
- **HTTP có 28 frames** — đây là điểm quan tâm nhất! HTTP truyền dữ liệu plaintext, dễ extract
- `image-jfif` — có ít nhất 3 ảnh JPEG được truyền qua HTTP
- SSDP/mDNS là traffic background bình thường của hệ điều hành (Spotify, Apple Bonjour...)

<img width="1919" height="876" alt="image" src="https://github.com/user-attachments/assets/2b00611c-36d2-40bd-8a7b-a5b8dd112b58" />

Phát hiện nhiều data trên **http**

---

### BƯỚC 3 — Lọc và liệt kê HTTP requests

**Lệnh:**
```bash
tshark -r yousaidwhat.pcapng -Y "http" -T fields -e frame.number -e http.request.method -e http.request.uri -e http.response.code

```

**Giải thích:**
- `-Y "http"` — **Y**=display filter, chỉ hiện packet HTTP 
- `-T fields` — output dạng fields thay vì full packet dump
- `-e frame.number` — lấy field số frame
- `-e http.request.method` — lấy method (GET, POST...)
- `-e http.request.uri` — lấy đường dẫn URL
- `-e http.response.code` — lấy HTTP status code (200, 404...)

**Kết quả:**
```
32    GET    /decoy2.txt
36               404
45    GET    /decoy1.txt
48               404
...  (lặp 10 lần decoy1 & decoy2 đều 404)
152   GET    /whoareyoucalling.zip
156               200
228   GET    /whossliming.jpg
261               200
279   GET    /stinky.jpeg
302               200
325   GET    /chicken.jpg
361               200
```

<img width="1919" height="874" alt="image" src="https://github.com/user-attachments/assets/496f6cc7-2913-4c48-9163-27e733c5ccd5" />

---

Tôi sẽ **export** để lấy các file này ra:

**Lệnh:**
```bash
mkdir -p extracted
tshark -r yousaidwhat.pcapng --export-objects http,extracted
ls -lh extracted/
```

<img width="1919" height="868" alt="image" src="https://github.com/user-attachments/assets/2b6f04fa-8663-41cc-9dd0-98ee59f1df63" />


**Kết quả:**
```
-rw-r--r-- chicken.jpg         134K
-rw-r--r-- decoy1.txt          335   (nhiều bản copy)
-rw-r--r-- decoy2.txt          335
-rw-r--r-- stinky.jpeg          40K
-rw-r--r-- whoareyoucalling.zip 243
-rw-r--r-- whossliming.jpg      77K
```

---

Thử xem cái file có tên giống tên bài này xem nó có gì:

**Lệnh:**
```bash
unzip -l extracted/whoareyoucalling.zip
```

**Kết quả:**
```
Archive:  whoareyoucalling.zip
  Length      Date    Time    Name
---------  ---------- -----   ----
       25  2025-10-30 03:51   whoareyoucalling.txt
---------                     -------
       25                     1 file
```

Thấy có 1 file .txt, thử giải nén ra xem

**Thử giải nén:**
```bash
unzip extracted/whoareyoucalling.zip
# → skipping: whoareyoucalling.txt  unable to get password
```

Cái file này nó cần **pass** để mở, tìm mấy file ảnh xem có khóa khum

---

Sau 1 thời gian lục lọi tôi đã tìm đc manh mối như bên dưới:


```
File Name           : chicken.jpg
Orientation         : Horizontal (normal)
Copyright           : 6e6f626f64792063616c6c73206d6520636869636b656e21
Profile Description : Adobe RGB (1998)
Image Width         : 1440
Image Height        : 922
```

1 chuỗi HEX kì lạ

```
Copyright           : 6e6f626f64792063616c6c73206d6520636869636b656e21
```

---

Tôi dùng cybercher để giải mã và đc chuỗi kì lạ:
<img width="1544" height="528" alt="image" src="https://github.com/user-attachments/assets/98ff87cf-17ed-49bc-b927-dedb89b178b5" />

```
nobody calls me chicken!
```

---

Có vẻ đây là **pass** để giải nén !

```bash
unzip -P 'nobody calls me chicken!' whoareyoucalling.zip 
```

**Output:**
```
Archive:  whoareyoucalling.zip
 extracting: whoareyoucalling.txt
```

Flag trong file này.

## Flag

```
UDCTF{wh4ts_wr0ng_mcf1y}
```

