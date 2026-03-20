# picoCTF - WPA-ing Out

## Overview
- **Category:** Forensics  
- **Difficulty:** Medium  

---

## Phân tích file với Wireshark

Mở file:
```bash
wireshark wpa-ing_out.pcap
```

### 🔎 Filter:

```bash
wlan
```
→ Xác nhận đây là **802.11 (WiFi traffic)**

```bash
eapol
```
→ Tìm **EAPOL packets (4-way handshake)**

<img width="1919" height="906" alt="image" src="https://github.com/user-attachments/assets/a66439cc-4d41-4514-afd8-a81b566bf20b" />


Nếu thấy EAPOL → có handshake → có thể crack WPA

---

## Nhận dạng

- WPA không gửi password dạng plaintext
- Chỉ có thể crack bằng brute-force/dictionary

=> Sử dụng **wordlist attack**

---

## Sử dụng aircrack-ng


```bash
aircrack-ng -w rockyou.txt wpa-ing_out.pcap
```

---

| Option | Ý nghĩa |
|------|--------|
| `aircrack-ng` | Tool crack WPA/WPA2 |
| `-w` | Chỉ định wordlist |
| `rockyou.txt` | Danh sách password phổ biến |
| `.pcap` | File chứa handshake |

---

## Kết quả

Sau khi chạy:
```
                               Aircrack-ng 1.7 

      [00:00:00] 1179/10303727 keys tested (2967.70 k/s) 

      Time left: 57 minutes, 51 seconds                          0.01%

                          KEY FOUND! [ mickeymouse ]


      Master Key     : 61 64 B9 5E FC 6F 41 70 70 81 F6 40 80 9F AF B1 
                       4A 9E C5 C4 E1 67 B8 AB 58 E3 E8 8E E6 66 EB 11 

      Transient Key  : 90 63 ED C6 BB 8A 59 D1 A5 E8 B4 6F 2F 89 66 C2 
                       0B D4 FC 62 37 2F 54 3B 7B B4 43 9B 37 F4 57 40 
                       FD D1 91 86 7F FE 26 85 7B AC DD 2C 44 E6 06 18 
                       03 B0 0F F2 75 A2 32 63 F7 35 74 2D 18 10 1C 25 

      EAPOL HMAC     : 65 2F 6C 0E 75 F0 49 27 6A AA 6A 06 A7 24 B9 A9 
```

---

## Flag


```
picoCTF{mickeymouse}
```


---

