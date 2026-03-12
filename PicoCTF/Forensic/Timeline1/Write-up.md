# Write-up: picoCTF -- Timeline 1 (Forensics)

## 1. Thông tin thử thách

-   **Tên bài:** Timeline 1\
-   **Thể loại:** Forensics (Pháp chứng kỹ thuật số)\
-   **Mục tiêu:** Tìm **flag** được ẩn trong file image của ổ đĩa bằng
    kỹ thuật **Timeline Analysis**.
-    [Link](https://challenge-files.picoctf.net/c_plain_mesa/fef9e3937fced503da228c6affaea69ed51d6234ed8fde14a52b573777b869e7/partition4.img.gz)

------------------------------------------------------------------------

<img width="956" height="529" alt="image" src="https://github.com/user-attachments/assets/2c413f32-0a21-4f00-a82d-336048245783" />

## 2. Tư duy hướng giải

### Trường hợp có Hint (Gợi ý từ tác giả)

1.  **Tạo Timeline**
    -   Sử dụng bộ công cụ **Sleuthkit** để xây dựng dòng thời gian của
        các hoạt động trên hệ thống.
2.  **Lọc các tệp mới**
    -   Dựa vào các trường **MACB**:
        -   **M** -- Modified
        -   **A** -- Accessed
        -   **C** -- Changed
        -   **B** -- Birth (Created)
3.  **Tìm dấu vết Anti-Forensic**
    -   Các lệnh thường dùng để xóa dấu vết:
        -   `shred`
        -   `rm`
        -   xóa `.bash_history`

    Những dấu vết này thường xuất hiện **ngay sau khi tạo file chứa
    flag**.

------------------------------------------------------------------------

### Trường hợp không có Hint (Tư duy thực tế của Forensics)

Quy trình chuẩn khi phân tích một ổ đĩa:

1.  **Xác định cấu trúc ổ đĩa**

``` bash
mmls partition4.img
```

hoặc

``` bash
fdisk -l partition4.img
```

2.  **Xây dựng Timeline hệ thống**
    -   Mục tiêu: xác định **chuyện gì đã xảy ra và vào lúc nào**.
3.  **Tìm các điểm bất thường**
    -   File nằm ở vị trí không phổ biến (ví dụ: file text trong `/etc`)
    -   File ẩn
    -   File trong thư mục người dùng `/root` hoặc `/home`
    -   File có tên đáng ngờ
4.  **Truy xuất trực tiếp bằng Inode**
    -   Tránh mount ổ đĩa để không làm thay đổi metadata.

------------------------------------------------------------------------

# 3. Các bước giải chi tiết

## Bước 1: Trích xuất Metadata (Body File)

Sử dụng `fls` để quét toàn bộ hệ thống tập tin.

``` bash
fls -r -m / partition4.img > timeline.body
```

### Giải thích

  Option   Ý nghĩa
  -------- ------------------------------------------
  `fls`    Liệt kê file và thư mục trong disk image
  `-r`     Quét đệ quy toàn bộ thư mục
  `-m /`   Tạo **body file** với mount point là `/`

File kết quả: **timeline.body**

------------------------------------------------------------------------

## Bước 2: Chuyển đổi thành Timeline dễ đọc

Sử dụng `mactime` để chuyển dữ liệu raw thành timeline.

``` bash
mactime -b timeline.body > timeline.txt
```

### Giải thích

  Option      Ý nghĩa
  ----------- ----------------------------
  `mactime`   Tạo timeline từ metadata
  `-b`        Chỉ định file body đầu vào

Kết quả thu được: **timeline.txt**

------------------------------------------------------------------------

## Bước 3: Phân tích Timeline

Tìm các file mới được tạo.

``` bash
grep "macb" timeline.txt | tail -n 20
```

### Kết quả

```
Tue Dec 02 2025 04:50:07     3072 m.c. d/drwxr-xr-x 0        0        32385    /etc
                               49 macb r/rrw-r--r-- 0        0        32716    /etc/chat
Tue Dec 02 2025 04:50:19     1024 .a.. d/drwx------ 0        0        172      /root
Tue Dec 02 2025 04:50:20       12 .a.. l/lrwxrwxrwx 0        0        64902    /usr/bin/shred -> /bin/busybox
```
Phát hiện file:

    /etc/chat
    inode: 32716

File này được tạo **ngay trước khi lệnh `shred` được thực thi**, cho
thấy có dấu hiệu **anti-forensics**.

------------------------------------------------------------------------

## Bước 4: Truy xuất dữ liệu bằng Inode

Thay vì đọc file theo đường dẫn, ta truy xuất trực tiếp từ **inode**.

``` bash
icat partition4.img 32716
```

### Giải thích

  Lệnh     Ý nghĩa
  -------- --------------------------------------------
  `icat`   Trích xuất dữ liệu file dựa trên **inode**

### Output

    NTczNDE3aDEzcl83aDRuXzdoM18xNDU3XzU4NTI3YmIyMjIK

------------------------------------------------------------------------

## Bước 5: Giải mã Base64

Chuỗi trên là **Base64**.

``` bash
echo "NTczNDE3aDEzcl83aDRuXzdoM18xNDU3XzU4NTI3YmIyMjIK" | base64 -d
```

### Kết quả

    573417h13r_7h4n_7h3_1457_58527bb222

------------------------------------------------------------------------

# 4. Tổng kết công cụ

  Công cụ     Option      Ý nghĩa
  ----------- ----------- ------------------------------------
  `fls`       `-r`        Quét toàn bộ hệ thống tập tin
              `-m`        Tạo **body file** cho mactime
  `mactime`   `-b`        Tạo timeline từ body file
  `icat`      `(inode)`   Trích xuất file trực tiếp từ inode
  `base64`    `-d`        Giải mã Base64

------------------------------------------------------------------------

# 5. Flag

    picoCTF{573417h13r_7h4n_7h3_1457_58527bb222}

------------------------------------------------------------------------

# 6. Kiến thức rút ra

-   **Timeline Analysis** là kỹ thuật cực kỳ quan trọng trong Digital
    Forensics.
-   Metadata của file có thể tiết lộ:
    -   thời gian tạo
    -   chỉnh sửa
    -   truy cập
-   **Anti-forensics** (ví dụ `shred`) thường để lại dấu vết trong
    timeline.
-   Có thể truy xuất file **đã bị xóa** bằng cách đọc trực tiếp
    **inode**.

------------------------------------------------------------------------
