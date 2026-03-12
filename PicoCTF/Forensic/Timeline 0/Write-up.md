# Write-up: picoCTF – Timeline 0

## 1. Thông tin bài toán

| Trường | Nội dung |
|--------|----------|
| **Tên bài** | Timeline 0 |
| **Thể loại** | Forensics |
| **Mô tả** | Tìm flag trong một file ảnh đĩa (disk image) |
| **Gợi ý** | *"Sloppy timestomping can yield strange (very old) timestamps"* |
---

<img width="954" height="534" alt="image" src="https://github.com/user-attachments/assets/d12ca619-4c44-4816-bf21-fe7efd47d0f9" />


---

## 2. Ý đồ bài toán & Tư duy giải quyết

### Ý đồ của tác giả

Tác giả muốn kiểm tra kỹ năng phân tích **siêu dữ liệu (metadata)** của hệ thống tập tin. Cụ thể là kỹ thuật **Timestomping** — một hành vi thường thấy ở kẻ tấn công nhằm **che giấu các file mã độc** bằng cách sửa ngày tạo/sửa đổi về quá khứ để không bị phát hiện khi điều tra viên tìm kiếm các file "vừa mới tạo".


---

## 3. Các bước giải chi tiết & Công cụ

### Bước 1: Phân tích cấu trúc và liệt kê Timeline

Sử dụng bộ công cụ **The Sleuth Kit (TSK)**, cụ thể là lệnh `fls`.

```bash
fls -r -m / partition4.img | sort -n -k 3 | head -n 20
```

**Giải thích các option:**

| Option | Ý nghĩa |
|--------|---------|
| `-r` | *(Recursive)* Quét đệ quy qua tất cả các thư mục con |
| `-m /` | Xuất theo định dạng **body file** với đường dẫn gốc là `/` |
| `sort -n -k 3` | Sắp xếp theo số (`-n`) ở cột thứ 3 – cột chứa Unix Timestamp |
| `head -n 20` | Chỉ lấy 20 dòng đầu tiên (file có thời gian cũ nhất) |

**Kết quả thu được:**

```
0|/bin/bbsuid|310|r/r--s--x--x|0|0|14248|1764621660|1754412272|1762891279|1762891279
0|/bin/bcab|4945|r/rrw-r--r--|0|0|41|473446800|473446800|473446800|473446800
0|/bin/busybox|23|r/rrwxr-xr-x|0|0|808712|1764621660|1754412272|1762891278|1762891278
0|/bin/cat -> /bin/busybox|36|l/lrwxrwxrwx|0|0|12|1764621661|1762891278|1762891278|1762891278
```

> Một file lạ tại `/bin/bcab` có **inode 4945** mang mốc thời gian `473446800`  
> (tương đương **năm 1985**), trong khi toàn bộ hệ thống là năm 2024.

---

### Bước 2: Trích xuất nội dung từ Inode

Sau khi xác định file nghi vấn, dùng lệnh `icat` để đọc nội dung bên trong.

```bash
icat partition4.img 4945
```

**Giải thích các option:**

| Option | Ý nghĩa |
|--------|---------|
| `icat` | Trích xuất nội dung file dựa trên số **Inode** thay vì đường dẫn |
| `4945` | Số inode đã tìm thấy ở Bước 1 |

**Kết quả thu được:**

```
NzFtMzExbjNfMHU3MTEzcl9oM3JfNDNhMmU3YWYK
```

---

### Bước 3: Giải mã kết quả (Decode)

Chuỗi ký tự trên có đặc điểm của **mã hóa Base64** (kết hợp chữ hoa, chữ thường và số).

Decode:

```bash
echo "NzFtMzExbjNfMHU3MTEzcl9oM3JfNDNhMmU3YWYK" | base64 -d
```


---

## 4. Kết quả

```
picoCTF{71m311n3_0u7113r_h3r_43a2e7af}
```

---

