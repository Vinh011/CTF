# Write-up: Sneaky Patch (TryHackMe)

## 1. Tổng quan bài

Bài **Sneaky Patch** thuộc dạng **Linux Forensics / Privilege
Escalation**.\
Mục tiêu của bài là tìm ra một **kernel module đáng ngờ** đã được cài
vào hệ thống và từ đó trích xuất **flag**.

Ý tưởng chính của bài:

-   Kiểm tra các **kernel module** đang chạy trong hệ thống.
-   Phát hiện một module lạ có tên **spatch**.
-   Phân tích file module `.ko`.
-   Tìm chuỗi bí mật được **encode dưới dạng hex**.
-   Decode hex để lấy flag.

------------------------------------------------------------------------

# 2. Kiểm tra kernel modules

Trong Linux, các kernel module có thể được liệt kê bằng lệnh:

``` bash
lsmod
```

Lệnh này hiển thị danh sách các module đang được nạp vào kernel.

Ví dụ kết quả:

    Module                  Size  Used by
    spatch                 16384  0

Ở đây ta thấy module **spatch**.

Đây không phải module chuẩn của Linux → dấu hiệu rất đáng nghi.

------------------------------------------------------------------------

# 3. Kiểm tra thông tin module

Ta dùng lệnh:

``` bash
modinfo spatch
```

Lệnh này hiển thị thông tin chi tiết của module:

-   author
-   description
-   filename
-   license

Kết quả cho thấy module này có mô tả liên quan đến việc patch hệ thống.

Điều này gợi ý đây có thể là **backdoor kernel module**.

------------------------------------------------------------------------

# 4. Tìm file module

Kernel module thường nằm ở:

    /lib/modules/<kernel-version>/

Ta có thể tìm file `.ko` bằng:

``` bash
find / -name "spatch.ko" 2>/dev/null
```

Ví dụ:

    /lib/modules/5.x.x/spatch.ko

------------------------------------------------------------------------

# 5. Phân tích module

Kernel module thực chất là **binary file**.

Ta có thể tìm các chuỗi văn bản bên trong bằng lệnh:

``` bash
strings spatch.ko
```

Sau khi chạy, ta thấy chuỗi đáng chú ý:

    Here's the secret: 54484d7b73757033725f736e33346b795f643030727d0a

Chuỗi phía sau là **hexadecimal**.

------------------------------------------------------------------------

# 6. Decode chuỗi hex

Ta dùng lệnh:

``` bash
echo 54484d7b73757033725f736e33346b795f643030727d0a | xxd -r -p
```

**-r**: hex -> binary

Kết quả:

    THM{sup3r_sn34ky_d00r}

------------------------------------------------------------------------

# 7. Flag

    THM{sup3r_sn34ky_d00r}

------------------------------------------------------------------------

# 8. Kiến thức rút ra

Bài này giúp hiểu:

-   Cách kiểm tra **kernel module** trong Linux
-   Cách phát hiện **module độc hại**
-   Phân tích **.ko kernel object**
-   Dùng `strings` để tìm dữ liệu ẩn trong binary
-   Decode dữ liệu hex

Các tool sử dụng:

-   lsmod
-   modinfo
-   find
-   strings
-   xxd

------------------------------------------------------------------------

# 9. Phân loại bài CTF

Bài này thuộc:

**Forensics + Linux Persistence + Kernel Module Analysis**

Trong thực tế, attacker có thể:

-   Cài **rootkit kernel module**
-   Ẩn tiến trình
-   Ẩn file
-   Backdoor hệ thống

Do đó việc kiểm tra kernel module là một bước quan trọng trong **Digital
Forensics**.
