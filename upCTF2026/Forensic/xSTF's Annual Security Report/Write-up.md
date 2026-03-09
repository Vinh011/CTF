# Write-up: xSTF's Annual Security Report

## Thông tin challenge

- **Tên bài**: `xSTF's Annual Security Report`
- **Category**: Forensics
- **File được cung cấp**: `2025-Security-Report.pdf`
- **Flag**: `upCTF{V3ry_b4d_S3cUriTy_P0stUr3}`

---

## Mô tả đề bài

Challenge cung cấp một file PDF có tên `2025-Security-Report.pdf` cùng mô tả đại ý:

> Tôi nhận được một bản báo cáo bảo mật bí mật của tổ chức mình, nhưng nhìn qua thì không thấy gì quá quan trọng. Tôi chia sẻ cho bạn với hy vọng bạn có thể giúp cải thiện bảo mật.

Đọc mô tả này có thể đoán ngay đây là kiểu bài **forensics giấu dữ liệu trong tài liệu**.  
Với các file PDF kiểu này, những hướng kiểm tra quan trọng thường là:

1. Kiểm tra nội dung hiển thị bình thường.
2. Kiểm tra metadata của PDF.
3. Kiểm tra xem PDF có **file đính kèm / object ẩn / stream lạ** hay không.
4. Nếu có file nhúng, tiếp tục phân tích file đó.
5. Nếu file nhúng bị mã hóa, tìm cách crack password.

---

## Bước 1: Kiểm tra file PDF ban đầu

Mở `2025-Security-Report.pdf` bình thường thì chỉ thấy một báo cáo bảo mật nhìn khá vô hại, không có flag hiện ra trực tiếp.

Có thể dùng một số lệnh cơ bản để thu thập thông tin:

```bash
file 2025-Security-Report.pdf
pdfinfo 2025-Security-Report.pdf
strings 2025-Security-Report.pdf | less
```

Ý tưởng ở đây là xác nhận:

- file đúng là PDF hợp lệ,
- số trang không nhiều,
- không có flag lộ rõ trong text,
- từ đó chuyển sang hướng tìm **dữ liệu ẩn**.

---

## Bước 2: Tìm dấu hiệu có file nhúng trong PDF

Với PDF, một hướng rất hiệu quả là kiểm tra **embedded files / attachments**.

Có vài công cụ có thể dùng:

### Cách 1: dùng `pdfdetach`
```bash
pdfdetach -list 2025-Security-Report.pdf
```

Nếu PDF có file nhúng, lệnh này thường sẽ liệt kê ra tên file đính kèm.

### Cách 2: dùng `binwalk`
```bash
binwalk -e 2025-Security-Report.pdf
```

`binwalk` khá hữu ích để phát hiện và trích xuất các thành phần nhúng bên trong file.

### Cách 3: dùng Python / thư viện PDF
Có thể dùng `pypdf` để kiểm tra attachment nếu muốn phân tích thủ công.

---

## Bước 3: Extract file nhúng

Sau khi kiểm tra, phát hiện trong `2025-Security-Report.pdf` có một file nhúng tên là:

```text
appendix.pdf
```

Đây chính là điểm bất thường quan trọng nhất của challenge.

Có thể extract bằng `pdfdetach`:

```bash
pdfdetach -saveall 2025-Security-Report.pdf
```

hoặc nếu dùng `binwalk`:

```bash
binwalk -e 2025-Security-Report.pdf
```

Sau khi extract xong, ta thu được file:

```text
appendix.pdf
```

---

## Bước 4: Phân tích `appendix.pdf`

Thử mở `appendix.pdf` thì phát hiện file này **không đọc được trực tiếp** vì đã bị đặt mật khẩu.

Dùng `pdfinfo` hoặc `qpdf` có thể xác nhận trạng thái mã hóa:

```bash
pdfinfo appendix.pdf
qpdf --show-encryption appendix.pdf
```

Kết quả cho thấy đây là một file PDF có encryption chuẩn của PDF, dạng:

- **Standard PDF encryption**
- **128-bit**
- **Revision 3**

Điều này có nghĩa là bước tiếp theo là **crack password** của file PDF này.

---

## Bước 5: Crack password của `appendix.pdf`

### 5.1 Tạo hash cho John the Ripper

Muốn dùng John the Ripper, trước tiên cần convert file PDF sang dạng hash mà John hiểu được.

Lệnh đúng là:

```bash
python3 /usr/share/john/pdf2john.py appendix.pdf > hash.txt
```

> Lưu ý:  
> Nếu gõ `pdf2john.py` mà báo `command not found` thì nguyên nhân là script không nằm trong PATH.  
> Trên Kali, thường có thể gọi trực tiếp bằng đường dẫn:
>
> ```bash
> python3 /usr/share/john/pdf2john.py appendix.pdf > hash.txt
> ```

Kiểm tra nội dung file hash:

```bash
cat hash.txt
```

Nó sẽ có một dòng bắt đầu bằng `$pdf$...`

---

### 5.2 Thử wordlist thông thường

Đầu tiên có thể thử `rockyou.txt`:

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

Trong quá trình làm bài, cách này **không crack ra ngay**.

---

### 5.3 Dùng `--rules`

Sau đó sử dụng John với tùy chọn `--rules`:

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt --rules hash.txt
```

`--rules` cho phép John không chỉ thử nguyên văn từng từ trong wordlist, mà còn tạo ra nhiều biến thể như:

- viết hoa chữ đầu,
- thêm số,
- thêm ký tự đặc biệt,
- các kiểu biến đổi phổ biến khác.

Ví dụ `maki` trong wordlist có thể được biến thành `Maki`, và đây chính là thứ giúp crack được password.

---

## Bước 6: Tìm ra mật khẩu

Sau khi chạy:

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt --rules hash.txt
```

John trả về kết quả:

```text
Maki             (appendix.pdf)
```

Có thể kiểm tra lại bằng:

```bash
john --show --format=PDF hash.txt
```

Vậy mật khẩu của `appendix.pdf` là:

```text
Maki
```

---

## Bước 7: Giải mã file PDF

Sau khi có password, giải mã file bằng `qpdf`:

```bash
qpdf --password='Maki' --decrypt appendix.pdf out.pdf
```

Sau đó trích text:

```bash
pdftotext out.pdf -
```

Hoặc đọc trực tiếp bằng Python:

```bash
python3 - << 'PY'
from pypdf import PdfReader

r = PdfReader("appendix.pdf")
r.decrypt("Maki")

for p in r.pages:
    print(p.extract_text())
PY
```

---

## Bước 8: Lấy flag

Sau khi mở được `appendix.pdf`, nội dung bên trong cho ra flag:

```text
upCTF{V3ry_b4d_S3cUriTy_P0stUr3}
```

---

# Flag

```text
upCTF{V3ry_b4d_S3cUriTy_P0stUr3}
```

---

## Tóm tắt hướng giải

Toàn bộ hướng giải của challenge này là:

1. Nhận file `2025-Security-Report.pdf`
2. Kiểm tra và phát hiện PDF có **file nhúng**
3. Extract file nhúng `appendix.pdf`
4. Phát hiện `appendix.pdf` bị khóa bằng password
5. Dùng `pdf2john.py` tạo hash
6. Dùng `john --wordlist=rockyou.txt --rules` để crack
7. Thu được password là `Maki`
8. Giải mã `appendix.pdf`
9. Đọc nội dung và lấy flag

---

## Vì sao bài này hay

Challenge này khá điển hình cho tư duy forensic:

- File “nhìn bình thường” nhưng thực tế có **attachment ẩn**
- Attachment lại là một **lớp bảo vệ thứ hai** bằng mật khẩu
- Password không hiện ra ngay với wordlist cơ bản, nhưng lại crack được khi bật **rules**

Bài nhắc lại một kỹ năng rất quan trọng:  
**Đừng chỉ nhìn nội dung hiển thị của PDF, hãy luôn kiểm tra object/attachment/metadata/file nhúng.**

---

## Một số lệnh hữu ích cho các bài PDF forensic tương tự

### Liệt kê attachment
```bash
pdfdetach -list file.pdf
```

### Extract attachment
```bash
pdfdetach -saveall file.pdf
```

### Xem thông tin PDF
```bash
pdfinfo file.pdf
```

### Kiểm tra mã hóa
```bash
qpdf --show-encryption file.pdf
```

### Tạo hash cho John
```bash
python3 /usr/share/john/pdf2john.py file.pdf > hash.txt
```

### Crack với wordlist
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

### Crack với rules
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt --rules hash.txt
```

### Xem password đã crack
```bash
john --show --format=PDF hash.txt
```

### Giải mã PDF
```bash
qpdf --password='PASSWORD' --decrypt file.pdf out.pdf
```

---

## Kết luận

Đây là một bài forensic khá thẳng hướng nhưng dễ mắc bẫy nếu chỉ đọc nội dung PDF bằng mắt thường.

Điểm mấu chốt là:

- **PDF gốc chỉ là mồi**
- **flag nằm trong attachment `appendix.pdf`**
- **`appendix.pdf` được bảo vệ bằng password**
- **password là `Maki`**
- **flag cuối cùng là `upCTF{V3ry_b4d_S3cUriTy_P0stUr3}`**

---

## Solve script ngắn gọn

```bash
pdfdetach -saveall 2025-Security-Report.pdf
python3 /usr/share/john/pdf2john.py appendix.pdf > hash.txt
john --wordlist=/usr/share/wordlists/rockyou.txt --rules hash.txt
qpdf --password='Maki' --decrypt appendix.pdf out.pdf
pdftotext out.pdf -
```

---

## Final Answer

```text
upCTF{V3ry_b4d_S3cUriTy_P0stUr3}
```
