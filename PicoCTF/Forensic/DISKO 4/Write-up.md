# 🚩 Write-up: picoCTF - DISKO 4

## 📖 Thông tin bài tập

-   **Thể loại:** Forensics\
-   **Mô tả:** Tìm flag trong một file disk image (`.dd`) đã bị xóa
    file.\
-   **Hint:** Làm thế nào để tìm kiếm các file đã bị xóa?
-   [link](https://challenge-files.picoctf.net/c_plain_mesa/ccb8edf69ba68d0d3dad5908c569e1306f75605b12327a94893e0f41bdd1597a/disko-4.dd.gz)

<img width="947" height="532" alt="image" src="https://github.com/user-attachments/assets/3bde79d8-febf-473c-ae60-f7553abc5b85" />

------------------------------------------------------------------------

# Tư duy hướng giải

Trong **Digital Forensics**, khi một file bị xóa trên hệ thống tệp (ví
dụ **FAT32 hoặc NTFS**), dữ liệu thực tế **không biến mất ngay lập
tức**.

### 1. Cấu trúc bảng tệp

Khi xóa file: - Hệ thống **không xóa dữ liệu ngay** - Chỉ **đánh dấu
entry là đã xóa**

Trong **FAT32**: - Ký tự đầu tiên của tên file bị thay bằng **`0xE5`**

### 2. Vùng dữ liệu

Các **cluster chứa nội dung file vẫn còn trên đĩa** cho đến khi:

-   Bị ghi đè bởi dữ liệu mới

### 3. Mục tiêu điều tra

Ta cần: - Tìm **metadata của file đã xóa** - Trích xuất dữ liệu từ
**cluster tương ứng**

------------------------------------------------------------------------

# 🛠 Công cụ sử dụng

  Tool       Mục đích
  ---------- ----------------------------------
  `file`     Xác định định dạng file
  `mmls`     Phân tích bảng phân vùng
  `fls`      Liệt kê file (kể cả file đã xóa)
  `icat`     Trích xuất dữ liệu theo inode
  `gunzip`   Giải nén file `.gz`

------------------------------------------------------------------------

# 👣 Các bước thực hiện

## Bước 1: Nhận diện định dạng ổ đĩa

``` bash
file disko-4.dd
```

### Kết quả

    DOS/MBR boot sector ... FAT (32 bit)

➡ Image sử dụng **FAT32 filesystem**.

------------------------------------------------------------------------

# Bước 2: Tìm file đã bị xóa

``` bash
fls -r disko-4.dd
```

### Giải thích

  Option   Ý nghĩa
  -------- ----------------------
  `fls`    Liệt kê file
  `-r`     Quét toàn bộ thư mục

### Kết quả đáng chú ý
```
    + r/r * 532021:  dont-delete.gz
```
-   Dấu `*` = **file đã bị xóa**
-   Metadata address = **532021**

------------------------------------------------------------------------

# Bước 3: Khôi phục file

``` bash
icat disko-4.dd 532021 > recovered.gz
```

Sau bước này ta thu được:

    recovered.gz

------------------------------------------------------------------------

# Bước 4: Giải nén file

``` bash
gunzip recovered.gz
```

Sau khi giải nén:

    recovered

------------------------------------------------------------------------

# Bước 5: Lấy flag

``` bash
cat recovered
```

### Output

    picoCTF{d3l_d0n7_h1d3_w3ll_dea2cdae}

------------------------------------------------------------------------

# 🏁 Flag

    picoCTF{d3l_d0n7_h1d3_w3ll_dea2cdae}

------------------------------------------------------------------------
