# CTF Writeup — Red Hearing 🔴👂

---

<img width="621" height="545" alt="image" src="https://github.com/user-attachments/assets/7e9f04ee-63f6-4f37-98e3-532d35929ce3" />


## Tổng quan bài

> *"Use all the skills you know about steganography..."*

File đính kèm: `bluehen.jpg`

Tên bài **"🔴👂ing"** = **Red Hearing** (chơi chữ từ *Red Herring* + *Hearing*). Đây là hint quan trọng nhất của bài — có một **mồi nhử** (red herring) được thiết kế để đánh lạc hướng ta.

---


## Lấy gốc

### JPEG File Format
JPEG là định dạng ảnh nén lossy. Cấu trúc byte của file JPEG chuẩn:

```
FF D8 FF       ← SOI (Start Of Image) — Magic bytes bắt buộc
FF E0 xx xx    ← APP0 marker (JFIF header)
FF FE xx xx    ← Comment segment (tùy chọn)
...
FF D9          ← EOI (End Of Image) — Kết thúc file
```

Nếu thiếu `FF D8 FF` ở đầu, tool nào kiểm tra magic bytes sẽ không nhận ra đây là JPEG.

### Steganography là gì?
Steganography = nghệ thuật **ẩn thông tin** bên trong dữ liệu khác mà người nhìn thường không phát hiện.

Các kỹ thuật phổ biến:
- **LSB (Least Significant Bit):** Thay đổi bit cuối của mỗi pixel → ảnh trông như cũ nhưng chứa data ẩn
- **Steghide:** Tool phổ biến, ẩn data vào JPG/BMP bằng cách thay đổi tần số DCT
- **Binwalk:** Phát hiện file ẩn bên trong file khác (appended data)
- **Metadata hiding:** Nhét flag vào EXIF, comment field

---

## Giải từng bước

### Bước 0 — Nhận file và nhìn tổng quan

```bash
ls -la bluehen.jpg
# Output: 52748 bytes — file khá nhỏ
```

---

### Bước 1 Nhận dạng file
```bash
file bluehen.jpg
# Output: bluehen.jpg: data
```


### Bước 2 Xem hex dump

```bash
hexedit bluehen.jpg
```

**Output:**
```
00000000   E0 00 10 4A  46 49 46 00  01 01 01 00  48 00 48 00  00 FF FE 00  2A 56 55 52  ...JFIF.....H.H.....*VUR
00000018   44 56 45 5A  37 65 54 42  31 58 32 77  78 61 7A 4E  66 63 6A 4E  6B 58 32 67  DVEZ7eTB1X2wxazNfcjNkX2g
00000030   7A 4E 48 49  78 62 6A 59  2F 66 51 3D  3D FF DB 00  43 00 02 01  01 01 01 01  zNHIxbjY/fQ==...C.......
```

**Phân tích hex:**

| Offset | Bytes thực tế | Bytes đúng chuẩn JPEG |
|--------|--------------|----------------------|
| 000000 | `E0 00 10` | `FF D8 FF E0 00 10` |

→ **File bị mất 3 bytes đầu `FF D8 FF`** (SOI marker của JPEG)!

Tại offset `000010-000030` có chuỗi ASCII nhìn thấy rõ:
```
*VURDVEZ7eTB1X2wxazNfcjNkX2gzNHIxbjY/fQ==
```
Đây là chuỗi Base64 nằm trong **JPEG Comment field** (marker `FF FE`).

---

### Bước 3 Decode Base64

```bash
echo "VURDVEZ7eTB1X2wxazNfcjNkX2gzNHIxbjY/fQ==" | base64 -d
# Output: UDCTF{y0u_l1k3_r3d_h34r1n6?}
```

**Kết quả decode:** `UDCTF{y0u_l1k3_r3d_h34r1n6?}`

Flag sai

---

### Bước 4 Fix JPEG header bị corrupt

```python
python3 -c "
data = open('bluehen.jpg','rb').read()
fixed = bytes([0xFF, 0xD8, 0xFF]) + data
open('bluehen_fixed.jpg','wb').write(fixed)
print('Fixed! Size:', len(fixed))
"
```

Hoặc

```bash
printf '\xFF\xD8\xFF' | cat - bluehen.jpg > bluehen_fixed.jpg
```

**Giải thích:**  
Ta đọc file gốc vào memory, thêm 3 magic bytes `FF D8 FF` vào đầu, rồi ghi ra file mới.

**Kiểm tra lại:**
```bash
exiftool bluehen_fixed.jpg
# File Type: JPEG ✓
# Comment: VURDVEZ7eTB1X2wxazNfcjNkX2gzNHIxbjY/fQ==
```

Bây giờ exiftool nhận ra được file.

---


**Steghide:**  
Steghide là tool ẩn/trích xuất data từ file ảnh bằng cách thay đổi các hệ số DCT (Discrete Cosine Transform) trong ảnh JPEG. Data bị ẩn có thể được encrypt bằng password.


---

### Bước 6 — Dùng chuỗi Base64 làm password Steghide

```bash
steghide extract -sf bluehen_fixed.jpg \
    -p "VURDVEZ7eTB1X2wxazNfcjNkX2gzNHIxbjY/fQ==" \
    -f 2>&1

# Output: wrote extracted data to "VURDVEZ7cHIzNzd5IDUxbXBsMyBwcjBibDNtIG4wP30=.txt"
```

Extract thành công! File được tạo ra có tên... cũng là một chuỗi Base64!

---

### Bước 7 Đọc file và decode tên file
```bash
# Đọc nội dung file
cat "VURDVEZ7cHIzNzd5IDUxbXBsMyBwcjBibDNtIG4wP30=.txt"
# Output: UDCTF{l0l_y0u_607_7r1ck3d}

# Decode tên file (cũng là Base64!)
echo "VURDVEZ7cHIzNzd5IDUxbXBsMyBwcjBibDNtIG4wP30=" | base64 -d
# Output: UDCTF{pr377y 51mpl3 pr0bl3m n0?}
```

---
