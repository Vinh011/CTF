# Write-up: endianness-v2

## Tổng quan

Chúng ta được cung cấp một file trích xuất từ hệ thống 32-bit nhưng dữ
liệu bị lưu sai thứ tự byte.
[FILE](https://artifacts.picoctf.net/c_titan/115/challengefile)
------------------------------------------------------------------------

## hướng

### 1. Nhận diện

-   File không mở được → **header (magic bytes)** bị hỏng.
-   Đề nhắc tới "32-bit" → gợi ý xử lý theo **block 4 byte**.
-   "Cách kỳ lạ" → ám chỉ **endianness bị đảo ngược**.

\> Dữ liệu đã bị đảo từng cụm 4 byte (little ↔ big endian).

------------------------------------------------------------------------

### 2. Ý đồ bài

#### 📌 Endianness

-   **Little-endian**: byte thấp trước\
-   **Big-endian**: byte cao trước

Ví dụ:

    AB CD EF 12

→ bị đảo thành:

    12 EF CD AB

------------------------------------------------------------------------

#### 📌 Magic Bytes

-   Là các byte đầu file giúp nhận dạng định dạng (JPEG, PNG,...)
-   Khi bị đảo → hệ điều hành không nhận ra file

------------------------------------------------------------------------

## Giải

### Cách 1: Python

``` python
with open("challengefile", "rb") as f:
    data = f.read()

fixed_data = bytearray()

for i in range(0, len(data), 4):
    chunk = data[i:i+4]
    fixed_data.extend(chunk[::-1])

with open("fixed_file", "wb") as f:
    f.write(fixed_data)

print("Done!")
```

Ý tưởng: - Đọc file dạng nhị phân - Chia thành block 4 byte - Đảo
từng block - Ghép lại
- Khi chạy xong thì tạo ra file **fixed_file**
- Dùng lệnh **file** để xem loại file:
```
file fixed_file
```

Kết quả:

```
└─$ file fixed_file 
fixed_file: JPEG image data, JFIF standard 1.01, aspect ratio, density 1x1, segment length 16, baseline, precision 8, 300x150, components 3
```

Phát hiện file có định dạng là **JPEG** nên đổi đôi file lại để mở file

```bash
mv fixed_file a.jpeg
```

------------------------------------------------------------------------

### Cách 2: CyberChef

-   Load file
-   Dùng:
    -   **Swap endianness**
    -   Word length: `4 bytes`
    -   Format: `Raw`
-   Export file
<img width="1919" height="920" alt="image" src="https://github.com/user-attachments/assets/4a158c49-75de-490e-8db1-a1a8fd934cc3" />

------------------------------------------------------------------------

## Flag

picoCTF{cert!f1Ed_iNd!4n_s0rrY_3nDian_004850bf}
