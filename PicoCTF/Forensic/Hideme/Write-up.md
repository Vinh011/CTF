# Write-up Hideme - picoCTF
**Mảng:** Forensic
**Loại:** Ẩn tin (Steganography)

---

# 1.Mô tả

Bài cung cấp một file ảnh:

---
flag.png
---

Khi mở ảnh bằng trình xem ảnh bình thường thi chỉ thấy **logo picoCTF** và **không có flag**.

Điều này gợi ý rằng:

- Có thể **flag được giấu trong ảnh**
- Hoặc **có dữ liệu khác được nhúng vào ảnh**

Đây là kỹ thuật thường gặp trong CTF gọi là:
**Steganography (Ẩn tin trong ảnh).**

---

# 2. Kiểm tra loại file

Sử dụng lệnh:

```bash
file flag.png
```

Output:

```
flag.png: PNG image data
```

Điều này cho thấy file đúng là **PNG image**.

Tuy nhiên trong CTF, file có thể chứa **dữ liệu ẩn phía sâu ảnh**.

---

# 3. Kiểm tra chuỗi trong file

Sử dụng:

```bash
strings flag.png
```

hoặc

```bash
string flag.png | grep -i flag
```

Có thể thấy các chuỗi đáng chú ý như:

```
secret/flag.png
```

Điều này cho thấy:
- có khả năng **bên trong file PNG có chứa file khác**
- Đặc biệt là một file **flag.png trong thư mục secret**

---

# 4. Phân tích hex của file

Sử dụng: 

```bash
xxd flag.png
```

hoặc mở rộng **hex editor**.

Khi xem phần cuối file sẽ thấy signature:

```
50 4B  05 06
```

ASCII của nó là:

```
PK
```

Đây là **magic number của file ZIP**.

Điều này chứng tỏ:

> Trong file PNG có **một file ZIP được nhúng vào phía sau**.

---

# 5. Phân tích bằng binwalk
Sử dụng công cụ forensic:
```bash
binwalk flag.png
```

Output:

```
DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             PNG image, 512 x 504, 8-bit/color RGBA, non-interlaced
41            0x29            Zlib compressed data, compressed
39739         0x9B3B          Zip archive data, at least v1.0 to extract, name: secret/
39804         0x9B7C          Zip archive data, at least v2.0 to extract, compressed size: 2959, uncompressed size: 3108, name: secret/flag.png
42998         0xA7F6          End of Zip archive, footer length: 22
```

Điều này cho thấy:
- Tại **offset 39804**
- Có **ZIP archive**

---

# 6. Extract dữ liệu ẩn

Sử dụng:
```bash
binwalk -e flag.png
```

Sau khi chạy, binwalk sẽ tạo thư mục:

```
├── flag.png
├── _flag.png.extracted
│   ├── 29
│   ├── 29.zlib
│   ├── 9B3B.zip
│   └── secret
│       └── flag.png
```

---
# 7. Lấy flag

Mở file:

```bash
eog flag.png
```

Hoặc:

```bash
xdg-open flag.png
```

**Flag:**
  picoCTF{Hiddinng_An_imag3_within_@n_ima9e_ad9f6587}
