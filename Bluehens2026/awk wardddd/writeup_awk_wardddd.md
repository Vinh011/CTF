# Write-up: awk...wardddd
<img width="611" height="588" alt="image" src="https://github.com/user-attachments/assets/a2a4c99e-c6fc-4b01-861d-c3ed28eb7d18" />

---

## Mô tả đề bài

> *"We recovered a directory from a misconfigured archive job. Most of the contents appear to be redundant or stale, but a few records still reflect the system's original processing format. Focus on what remains consistent."*

**Tên challenge: `awk...wardddd`** → hint trực tiếp sử dụng công cụ `awk` hoặc text processing để lọc dữ liệu.

---

## Giải

Kiểm tra xem file nó như nào, có đúng cấu trúc không ?

```bash
file s0rry_in_4dv4nc3.zip
# Output: Zip archive data, at least v2.0 to extract
```

---

Thử xem xem file nén có gì:

```bash
unzip -l s0rry_in_4dv4nc3.zip | head -50
```

**Output:**
```
Archive: s0rry_in_4dv4nc3.zip
  10018 files
  Thư mục: archive/, logs/, users/, tmp/, reports/
  File extension: .rec, .tmp, .dat, .log, .cache, .txt
```
---

Thấy nhiều file nên tôi giải nén ra xem có gì trong có, có flag k

```bash
unzip s0rry_in_4dv4nc3.zip  # -q = quiet mode, không spam output
ls s0rry_in_4dv4nc3/
# Output: archive  logs  reports  tmp  users
```

Thử xem 1 file xem:

```bash
cat s0rry_in_4dv4nc3/archive/9cDHTXAv03.rec
```

**Cấu trúc một file:**
```
timestamp=178087
profile=gamma
uid=5463
state=disabled
part=01
data=UZMO5D
note=cxYaHEZ7vVOc_jeA
comment=VURDVEZ7ZmFrZV9mbGFnfQ==
```

> Đa số các file đều có các trường này, tìm xem có gì khác biệt không ?

**Phân tích fields:**
| Field | Giá trị mẫu | Ý nghĩa |
|-------|------------|---------|
| `timestamp` | Random số | Timestamp giả |
| `profile` | alpha/beta/gamma/delta/omega | Nhóm/loại record |
| `uid` | Random số | User ID giả |
| `state` | active/archived/disabled/pending | **Trạng thái** |
| `part` | 01, 02, ... | **Số thứ tự mảnh** |
| `data` | Base64 string | **Dữ liệu thực** |
| `note` | Random string | Ghi chú |
| `comment` | Base64 string | Chú thích |


Phát hiện mỗi file có **part** và **state** nghi nghi:
- Các trạng thái **state**:
  - **active**: Đang hoạt động
  - **inactive**: Không còn hoạt động
  - **closed**: Đã đóng
  - **terminated**: Bị kill
  - **deleted**: Đã xóa

Đề bài gợi ý sử dụng tool **awk** nhưng tôi thấy nó phức tạp nên tôi dùng tool dễ hiểu hơn:

```bash
└─$ find . -type f -exec grep "^state=active" {} +
```
- Tìm tất cả file trong thư mục hiện tại, rồi **lọc ra những file có dòng bắt đầu bằng `state=active`**

**Output**:

```
./s0rry_in_4dv4nc3/archive/sys_eXAuo_05.rec:state=active
./s0rry_in_4dv4nc3/archive/sys_we9bk_04.rec:state=active
./s0rry_in_4dv4nc3/tmp/sys_t-Bfc_03.rec:state=active
./s0rry_in_4dv4nc3/tmp/sys_lYOTr_07.rec:state=active
./s0rry_in_4dv4nc3/logs/sys_gL6JX_02.rec:state=active
./s0rry_in_4dv4nc3/logs/sys_tBxJ4_06.rec:state=active
./s0rry_in_4dv4nc3/users/sys_FtSns_01.rec:state=active
```

Nhiền vào **output** tôi thấy đc các file có dòng là **state=active** và phát hiện file có đuôi từ 1 - 7, tôi thử mở xem nó có nội dung gì ?

---

Sau khi đọc 7 file đó tôi thấy đc các text trông có vẻ là base 64:

**Output:**
```
01 VURDVEZ7
02 dzNsbF83
03 aDQ3X3c0
04 NW4nN183
05 MF9oNHJk
06 X3c0NV8x
07 Nz99
```

```bash
# Ghép tất cả và decode một lần
echo "VURDVEVZdzNzbF83aDQ3X3c0NW4nN183MF9oNHJkX3c0NV8xNz99" | base64 -d
# → UDCTF{w3ll_7h47_w45n'7_70_h4rd_w45_17?}
```

FLAG: `UDCTF{w3ll_7h47_w45n'7_70_h4rd_w45_17?}`**
