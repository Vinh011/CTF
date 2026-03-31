# Write-up: picoCTF 2019 - c0rrupt (Forensics)

## 1. Tổng quan

-   **Tên bài**: c0rrupt\
-   **Thể loại**: Forensics\
-   **Mục tiêu**: Khôi phục file ảnh bị hỏng để lấy flag
<img width="953" height="639" alt="image" src="https://github.com/user-attachments/assets/369b30c0-95cf-40bd-a22b-7af7ac2520cb" />

------------------------------------------------------------------------

## 2. Phân tích ban đầu

File không nhận diện được:

    file mystery

→ Kết luận: file bị hỏng header

- Dùng lệnh kiểm tra file **png** và phát hiện ra nhiều lỗi:
```bash
  pngcheck -c -v mystery.png
```
Kết quả:
```
└─$ pngcheck -c -v mystery    
zlib warning:  different version (expected 1.2.13, using 1.3.1)

File: mystery (202940 bytes)
  this is neither a PNG or JNG image nor a MNG stream
ERRORS DETECTED in mystery

```
------------------------------------------------------------------------

## 3. Kiểm tra header

PNG chuẩn:

    89 50 4E 47 0D 0A 1A 0A

→ Sửa lại header

------------------------------------------------------------------------

## 4. Cấu trúc PNG

    [length][type][data][CRC]

-   length: độ dài dữ liệu\
-   type: tên chunk\
-   data: dữ liệu\
-   CRC: kiểm tra lỗi

------------------------------------------------------------------------

## 5. Fix lỗi

### Lỗi 1: Sai chunk IHDR

Sửa:

    C"DR → IHDR

```
49 48 44 52
```

### Lỗi 2: CRC pHYs

Sửa data:

    AA 00 16 25 → 00 00 16 25

### Lỗi 3: Sai length

Sửa:

    AA AA FF A5 → 00 00 FF A5

### Lỗi 4: Sai IDAT

Sửa:

    AB 44 45 54 → 49 44 41 54

------------------------------------------------------------------------

## 6. Kiểm tra

```
    └─$ pngcheck -c -v mystery.png 
zlib warning:  different version (expected 1.2.13, using 1.3.1)

File: mystery.png (202940 bytes)
  chunk IHDR at offset 0x0000c, length 13
    1642 x 1095 image, 24-bit RGB, non-interlaced
  chunk sRGB at offset 0x00025, length 1
    rendering intent = perceptual
  chunk gAMA at offset 0x00032, length 4: 0.45455
  chunk pHYs at offset 0x00042, length 9: 5669x5669 pixels/meter (144 dpi)
  chunk IDAT at offset 0x00057, length 65445
    zlib: deflated, 32K window, fast compression
  chunk IDAT at offset 0x10008, length 65524
  chunk IDAT at offset 0x20008, length 65524
  chunk IDAT at offset 0x30008, length 6304
  chunk IEND at offset 0x318b4, length 0
No errors detected in mystery.png (9 chunks, 96.3% compression).
```

→ No errors detected

------------------------------------------------------------------------

## 7. Kết quả

Mở ảnh:

    eog mystery.png

→ Flag:

    picoCTF{c0rrupt10n_1847995}

------------------------------------------------------------------------

