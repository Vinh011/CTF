# Writeup: Blast from the past

<img width="946" height="776" alt="image" src="https://github.com/user-attachments/assets/21d89c00-9633-4074-9c04-1d958900c812" />

---

## Tổng quan

Ảnh kỹ thuật số không chỉ chứa pixels — nó còn chứa **metadata ẩn (EXIF)** như ngày chụp, máy ảnh, GPS. Bài này yêu cầu **giả mạo toàn bộ timestamps** về mốc `1970:01:01 00:00:00.001+00:00` — tức **Unix Epoch + 1 millisecond**.

Thách thức nằm ở chỗ: ảnh được chụp bằng **Samsung**, và Samsung nhúng một timestamp độc quyền vào cuối file mà exiftool **không thể ghi đè** — buộc phải sửa thủ công ở cấp độ binary.

File được cho:

- `original.jpg`: [Tải](https://artifacts.picoctf.net/c_mimas/88/original.jpg)

---

## Bước 1: Đọc metadata — xem có bao nhiêu timestamp

```bash
exiftool original.jpg
```

**Tại sao:** Trước khi sửa phải biết file có bao nhiêu trường timestamp. Sót 1 trường là checker fail.

**Output:**
```
Modify Date              : 2023:11:20 15:46:23
Date/Time Original       : 2023:11:20 15:46:23
Create Date              : 2023:11:20 15:46:23
Sub Sec Time             : 703
Sub Sec Time Original    : 703
Sub Sec Time Digitized   : 703
Time Stamp               : 2023:11:21 03:46:21.420+07:00  ← Samsung độc quyền
```

**Phát hiện:** Có **7 trường** cần sửa. Trường cuối `Time Stamp` là của Samsung — đặc biệt hơn các trường còn lại.

---

## Bước 2: Hiểu timestamp cần đặt

```
1970:01:01 00:00:00.001+00:00
```

| Phần | Ý nghĩa |
|------|---------|
| `1970:01:01` | Unix Epoch — mốc thời gian gốc của mọi hệ thống Unix |
| `00:00:00` | 0 giờ 0 phút 0 giây |
| `.001` | 1 millisecond |
| `+00:00` | UTC timezone |

> Checker chấp nhận mọi timezone **miễn quy đổi về UTC tương đương** — ví dụ `1970:01:01 07:00:00.001+07:00` cũng hợp lệ.

---

## Bước 3: Sửa 6 tags EXIF bằng exiftool

```bash
exiftool \
  -ModifyDate="1970:01:01 00:00:00.001+00:00" \
  -DateTimeOriginal="1970:01:01 00:00:00.001+00:00" \
  -CreateDate="1970:01:01 00:00:00.001+00:00" \
  -SubSecTime="001" \
  -SubSecTimeOriginal="001" \
  -SubSecTimeDigitized="001" \
  -overwrite_original \
  original.jpg
```

**Giải thích từng flag:**

| Flag | Ý nghĩa |
|------|---------|
| `-ModifyDate=...` | Ghi đè trường chỉ định bằng giá trị mới |
| `-SubSecTime="001"` | Phần millisecond phải set riêng — không nằm trong AllDates |
| `-overwrite_original` | Ghi đè file gốc, không tạo file backup `_original` |

---

## Bước 4: Tại sao Samsung TimeStamp không sửa được bằng exiftool?

Chạy thử:
```bash
exiftool -samsung:TimeStamp="1" original.jpg
# Warning: Sorry, samsung:TimeStamp doesn't exist or isn't writable
```

**Lý do:** Samsung lưu timestamp theo cách riêng — không phải EXIF chuẩn mà là **chuỗi ASCII nhúng thẳng vào binary** ở cuối file JPEG, dạng:

```
Image_UTC_Data1700513181420
```

Trong đó `1700513181420` là **Unix milliseconds** — tức số milliseconds kể từ Epoch.

**Tại sao biết là milliseconds?**

- Con số có **13 chữ số** → quy ước: 10 chữ số = seconds, 13 chữ số = milliseconds
- Thử tính: `1700513181420 ÷ 1000 = 1700513181.420 giây` → convert ra `2023:11:20 20:46:21.420 UTC` — khớp chính xác với giá trị exiftool đọc được 

**Quy tắc nhận biết Unix timestamp:**

```
10 chữ số → seconds      (đến năm ~2038)
13 chữ số → milliseconds
16 chữ số → microseconds
```

**Giá trị cần đặt:**
```
1970:01:01 00:00:00.001+00:00 = 1 millisecond = 1
→ chuỗi 13 ký tự: 0000000000001
```

> Phải giữ **đúng 13 ký tự** để không làm hỏng cấu trúc file!

---

## Bước 5: Sửa Samsung TimeStamp — 3 cách

### Cách 1: Python (nhanh nhất, không cần cài thêm)

```bash
python3 -c "
data = open('original.jpg','rb').read()
data = data.replace(b'1700513181420', b'0000000000001')
open('original.jpg','wb').write(data)
print('Done!')
"
```

**Tại sao được:** Timestamp Samsung lưu dạng **ASCII string** trong binary — Python tìm và replace chuỗi bytes trực tiếp mà không cần tính hex.

---

### Cách 2: bvi — hex editor terminal

```bash
sudo apt install bvi
bvi original.jpg
```

Trong bvi (cú pháp giống vim):

```
/Image_UTC_Data    ← tìm chuỗi, nhấn Enter
14l                ← nhảy sang phải 14 ký tự (qua "Image_UTC_Data")
r0                 ← replace '1' → '0'
r0                 ← replace '7' → '0'
r0                 ← replace '0' → '0'
r0                 ← replace '0' → '0'
r0                 ← replace '5' → '0'
r0                 ← replace '1' → '0'
r0                 ← replace '3' → '0'
r0                 ← replace '1' → '0'
r0                 ← replace '8' → '0'
r0                 ← replace '1' → '0'
r0                 ← replace '4' → '0'
r0                 ← replace '2' → '0'
r1                 ← replace '0' → '1'  ← ký tự cuối!
:wq                ← lưu và thoát
```

> Dùng `r` (replace) **không** dùng `i` (insert) — insert sẽ làm lệch toàn bộ cấu trúc file!

---

### Cách 3: Bless / GHex — hex editor GUI

```bash
sudo apt install ghex     # GHex thay thế cho Bless trên Kali
ghex original.jpg
```

1. `Search` → `Find & Replace` (`Ctrl+H`)
2. Chọn chế độ **Text**
3. Find: `1700513181420`
4. Replace: `0000000000001`
5. Click Replace → `Ctrl+S`

---

## Bước 6: Kiểm tra lại

```bash
exiftool original.jpg | grep -i "date\|time\|timestamp"
```

Kết quả mong đợi:
```
Modify Date        : 1970:01:01 00:00:00
Date/Time Original : 1970:01:01 00:00:00
Create Date        : 1970:01:01 00:00:00
Sub Sec Time       : 001
Time Stamp         : 1970:01:01 00:00:00.001+00:00 
```

---

## Bước 7: Submit

```bash
# Gửi file lên server
nc -w 2 mimas.picoctf.net 58371 < original.jpg

# Check kết quả
nc mimas.picoctf.net 53328
```

**Giải thích lệnh `nc`:**

| Phần | Ý nghĩa |
|------|---------|
| `nc` | netcat — gửi/nhận dữ liệu qua TCP như "ống nước" mạng |
| `-w 2` | timeout 2 giây, tự đóng kết nối sau khi gửi xong |
| `< original.jpg` | đọc file và gửi toàn bộ qua kết nối TCP |

**Output khi thành công:**
```
Checking tag 7/7
Looking at Samsung: TimeStamp
Looking for '1970:01:01 00:00:00.001+00:00'
Found: 1970:01:01 00:00:00.001+00:00
Great job, you got that one!
You did it!
picoCTF{71m3_7r4v311ng_p1c7ur3_a4f2b526}
```

---

## Flag

```bash
 #picoCTF{71m3_7r4v311ng_p1c7ur3_a4f2b526}
```

