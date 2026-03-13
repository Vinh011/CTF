# Write-up: Rogue Tower - picoCTF

------------------------------------------------------------------------
<img width="951" height="560" alt="image" src="https://github.com/user-attachments/assets/6db5a93b-b803-41e0-85c5-bd5b76df24c7" />

[Link file](https://challenge-files.picoctf.net/c_plain_mesa/986fd133a4070d35bbf6f1aef9e24aebf411cc0a40af3ccb7416d776a39d2374/rogue_tower.pcap)

# 1. Phân tích đề bài

Các hint của bài:

-   Look for unauthorized test network broadcasts on **UDP port 55000**
-   Find the device that connected to the rogue tower using **HTTP
    User-Agent**
-   The encryption key is derived from the victim device's **IMSI**
-   The exfiltrated data is split across multiple **HTTP POST requests**

Từ đó ta có hướng:

1.  Tìm broadcast bất thường trên UDP 55000\
2.  Xác định rogue tower (CELL ID)\
3.  Tìm thiết bị kết nối tới tower đó\
4.  Lấy IMSI của thiết bị\
5.  Tìm các HTTP POST chứa dữ liệu bị đánh cắp\
6.  Ghép dữ liệu lại\
7.  Giải mã để lấy flag

------------------------------------------------------------------------

# 2. Phân tích bằng Wireshark

## 2.1 Tìm broadcast rogue tower

Mở file `.pcap` trong Wireshark.

Filter:

udp.port == 55000

<img width="1916" height="907" alt="image" src="https://github.com/user-attachments/assets/f0007742-1c96-4aa5-a143-6cf3f86f2d22" />


Follow UDP Steam ta thấy các broadcast:

CARRIER: Verizon PLMN=310410 CELLID=15606\
CARRIER: AT&T PLMN=310410 CELLID=15607

Nhưng có một dòng đáng ngờ:

UNAUTHORIZED-TEST-NETWORK PLMN=00101 CELLID=92058

📌 Đây chính là rogue tower.

**Kết luận:**

CELLID = 92058

------------------------------------------------------------------------

## 2.2 Tìm thiết bị kết nối rogue tower

Filter HTTP request:

http.request

<img width="1919" height="903" alt="image" src="https://github.com/user-attachments/assets/eca00d7a-b0eb-4cc4-959f-412713f39202" />


Xem header **User-Agent** của các request.

Phát hiện:

User-Agent: MobileDevice/1.0 (IMSI:310410955678402; CELL:15606)

Tìm request chứa:

CELL:92058

Ta thấy:

User-Agent: MobileDevice/1.0 (IMSI:310410308555787; CELL:92058)

📌 Thiết bị nạn nhân:

IMSI: 310410308555787\
IP: 10.100.50.122

------------------------------------------------------------------------

## 2.3 Theo dõi traffic của nạn nhân

Filter:

ip.addr == 10.100.50.122

Sau khi kết nối rogue tower, thiết bị gửi nhiều request:

POST /upload HTTP/1.1\
Host: 198.51.100.58

Đây là **server nhận dữ liệu bị đánh cắp**.

------------------------------------------------------------------------

## 2.4 Thu thập dữ liệu exfiltration

Filter hoặc Follow HTTP Stream ta thấy chuỗi base64 trong mỗi request:

ip.addr == 10.100.50.122 && http.request.method == "POST"

Body các POST:

QFFWWnZjf\
kxCCFJABm\
hbBFxUakE\
FQAtFb1xX\
VgEHAAQBR\
Q==

Ghép lại:

> QFFWWnZjfkxCCFJABmhbBFxUakEFQAtFb1xXVgEHAAQBRQ==

------------------------------------------------------------------------

# 3. Phân tích bằng tshark


------------------------------------------------------------------------

## 3.1 Tìm broadcast port 55000

tshark -r capture.pcap -Y "udp.port == 55000"

------------------------------------------------------------------------

## 3.2 Tìm User-Agent chứa CELLID

tshark -r capture.pcap -Y http.user_agent -T fields -e http.user_agent

Lọc CELLID rogue:

tshark -r capture.pcap -Y 'http.user_agent contains "92058"' -T fields
-e http.user_agent

Kết quả:

MobileDevice/1.0 (IMSI:310410308555787; CELL:92058)

------------------------------------------------------------------------

## 3.3 Tìm HTTP POST của nạn nhân

tshark -r capture.pcap -Y 'ip.addr==10.100.50.122 &&
http.request.method=="POST"'

------------------------------------------------------------------------

## 3.4 Trích xuất payload POST

tshark -r capture.pcap -Y 'http.request.method=="POST"' -T fields -e
data

Sau đó ghép các chunk lại thành chuỗi base64.

------------------------------------------------------------------------

# 4. Decode dữ liệu

Base64 decode:

echo "QFFWWnZjfkxCCFJABmhbBFxUakEFQAtFb1xXVgEHAAQBRQ==" \| base64 -d

Kết quả là dữ liệu đã mã hóa.

------------------------------------------------------------------------

# 5. Suy encryption key từ IMSI

IMSI của nạn nhân:

310410308555787

Key được lấy từ một phần của IMSI:

```
08555787
```

Ném **key** này lên Cyberchef để XOR với chuỗi **base64** đã được giải mã.

<img width="1917" height="923" alt="image" src="https://github.com/user-attachments/assets/0c48ce1d-6840-465f-b677-d484645009bb" />


------------------------------------------------------------------------


# 7. Flag

picoCTF{r0gu3_c3ll_t0w3r_dbc40831}

------------------------------------------------------------------------
