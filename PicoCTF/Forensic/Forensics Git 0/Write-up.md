
# 🚩 Write-up: picoCTF – Forensics Git 0

## 1. Phân tích 


### Lệnh
```bash
file disk.img
```

### Kết quả
```
DOS/MBR boot sector
```

### Ý nghĩa

Điều này cho thấy:

- File là **disk image có bảng phân vùng MBR**
- Bên trong có thể chứa **nhiều partition**

---

# 2. Trích xuất và Truy cập dữ liệu (Extraction)

Vì đây là **disk image**, ta cần mở nó để xem dữ liệu bên trong.

Có hai cách:

| Phương pháp | Mô tả |
|---|---|
| Mount | Gắn image vào hệ thống |
| Extract | Giải nén nội dung |



## Công cụ sử dụng

```
7z
```

### Option quan trọng

| Option | Ý nghĩa |
|---|---|
| `x` | Extract giữ nguyên cấu trúc |
| `-o` | Chỉ định thư mục output |

### Lệnh

```bash
7z x disk.img -oextracted
```

### Kết quả

7z sẽ tách các **partition image** ra:

```
0.img
1.img
2.img
```

---

# 3. Phân tích phân vùng khả nghi (Partition Analysis)

Sau khi có các partition, ta cần tìm **phân vùng chứa dữ liệu người dùng**.


Thông thường:

- **Phân vùng lớn nhất** chứa
  - hệ điều hành
  - thư mục `/home`
  - dữ liệu người dùng

Trong bài này:

```
2.img
```

là phân vùng đáng nghi nhất.

### Giải nén tiếp

```bash
7z x 2.img -oanalysis
```
- Giải nén với thư mục là **analysis**

Sau đó có thể duyệt thư mục bên trong.

---

# 4. Điều tra Git (Git Forensics)

Tên bài có chữ **Git**, nên mục tiêu là tìm **repository Git**.

Repository Git nằm trong thư mục ẩn:

```
./home/ctf-player/Code/secrets/.git
```

Thư mục `.git` chứa:

- toàn bộ lịch sử commit
- metadata
- các version cũ của file

---

# 5. Phân tích lịch sử Git

Sau khi tìm thấy repository:

### Xem commit history

```bash
git log
```

Kết quả:

```
└─$ git log   
commit 327681bb38cf467cec328eec9707b240e3e74ced (HEAD -> master)
Author: ctf-player <ctf-player@example.com>
Date:   Wed Nov 19 08:49:27 2025 +0000

    Wrap this phrase in the flag format: g17_1n_7h3_d15k_041217d8
    
```

---

# Flag

```
picoCTF{g17_1n_7h3_d15k_041217d8}
```

---

