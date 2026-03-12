
# 🚩 Write-up: Forensics Git 1 (picoCTF)

<img width="949" height="505" alt="image" src="https://github.com/user-attachments/assets/b34645a5-291b-43f5-a1c8-8f96dd56f694" />

---

[Link](https://challenge-files.picoctf.net/c_plain_mesa/4538dd1f2e93e907c17f0b663c0e1fae2d7054a72b4ee36977f20cfbf3b0a01c/disk.img.gz)

# 1. Phân tích tệp tin (File Analysis)

Chúng ta nhận được một file:

```
disk.img
```

Đây là **disk image**, có thể chứa nhiều partition hoặc filesystem bên trong.

### Kiểm tra và giải nén

Sử dụng **7z** để trích xuất nội dung.

```bash
7z x disk.img -oaaa
```

### Kết quả

Sau khi giải nén ta thấy:

```
0.img
1.img
2.img
```

Thông thường:

- partition nhỏ → boot
- partition lớn → dữ liệu hệ thống

Qua kiểm tra, **2.img** là phân vùng Linux chứa dữ liệu quan trọng.

---

# 2. Khai thác dữ liệu (Data Extraction)

Tiếp tục giải nén phân vùng nghi ngờ:

```bash
7z x 2.img -oanalysis
```

Sau khi giải nén, ta có toàn bộ filesystem.

---

# 3. Tìm Git Repository

Tên bài có chữ **Git**, nên mục tiêu là tìm **repository Git**.

### Tìm thư mục `.git`

```bash
find . -type d -name ".git"
```

### Kết quả

```
./analysis/home/ctf-player/Code/secrets/.git
```

Repository Git nằm tại:

```
analysis/home/ctf-player/Code/secrets
```

---

# 4. Điều tra Git (Git Forensics)

Di chuyển vào thư mục repository:

```bash
cd analysis/home/ctf-player/Code/secrets
```

Kiểm tra nội dung:

```bash
ls -la
```

Kết quả:

```
.git
```

Không có file source → có thể **flag đã bị xóa**.

---

# 5. Kiểm tra lịch sử commit

Git lưu **toàn bộ lịch sử thay đổi** của project.

### Xem commit history

```bash
git log
```

### Output

Ta thấy 2 commit:

```
Add flag
Remove flag
```

---

# 6. Xem nội dung bị thay đổi

Để xem chi tiết những thay đổi trong từng commit:

```bash
git log -p
```

Option:

| Option | Ý nghĩa |
|------|------|
| `-p` | Hiển thị diff của commit |

---

### Kết quả

```
└─$ git log -p
commit 5fb8194539c770a830b8ba089a50778c07072b03 (HEAD -> master)
Author: ctf-player <ctf-player@example.com>
Date:   Wed Nov 19 09:20:05 2025 +0000

    Remove flag

diff --git a/flag.txt b/flag.txt
deleted file mode 100644
index f150f47..0000000
--- a/flag.txt
+++ /dev/null
@@ -1 +0,0 @@
-picoCTF{g17_r3m3mb3r5_d4ddf904}
\ No newline at end of file

commit 177789af0b300e043ea8f54ea57d6cee352291ae
Author: ctf-player <ctf-player@example.com>
Date:   Wed Nov 19 09:20:05 2025 +0000

    Add flag

diff --git a/flag.txt b/flag.txt
new file mode 100644
index 0000000..f150f47
--- /dev/null
+++ b/flag.txt
@@ -0,0 +1 @@
+picoCTF{g17_r3m3mb3r5_d4ddf904}
\ No newline at end of file
```
---

# 🏁 Flag

```
picoCTF{g17_r3m3mb3r5_d4ddf904}
```

---


## Git Forensics

Git lưu:

- toàn bộ commit
- lịch sử file
- nội dung file đã bị xóa

Các lệnh quan trọng:

```
git log
git log -p
git show
git diff
```


