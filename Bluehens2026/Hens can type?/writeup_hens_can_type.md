# CTF Writeup: "Hens can type?" — USB Forensics

<img width="623" height="742" alt="image" src="https://github.com/user-attachments/assets/359784cc-fb65-46b6-9ddc-f7c9c6da1398" />

---

## 1. Tổng quan

### Đề bài

```
UD SOC team recovered a USB traffic capture from a suspicious machine on campus.
Investigators believe a user typed something important... Can you reconstruct what was typed?
Take a closer look you might find what was left behind.
```

File đính kèm: `challenge1.pcapng`

---

<img width="1919" height="912" alt="image" src="https://github.com/user-attachments/assets/bfc837af-fd7a-4d59-b66b-dfc379e98f3c" />

Data chủ yếu là USB, tôi lọc để xem thông tin gói tin: 

<img width="1919" height="869" alt="image" src="https://github.com/user-attachments/assets/c6fb264a-9206-4ed3-a36c-94254c7b3434" />

### Trong trường info tôi biết đây là bảng vẽ điện tử (Graphics Tablet).
- URB (USB Request Block): Là một cấu trúc dữ liệu (đơn vị vận chuyển) dùng để chứa thông tin giữa driver trên máy tính và thiết bị USB. Nó giống như một "toa tàu" chở dữ liệu.

- INTERRUPT (Ngắt): Đây là một trong bốn loại truyền tải của chuẩn USB (Control, Bulk, Isochronous, Interrupt). Đối với các thiết bị như bảng vẽ, chuột, bàn phím, dữ liệu không được gửi liên tục theo dòng mà chỉ gửi khi có sự thay đổi (như khi bạn di chuyển bút hoặc nhấn nút).

---

Khác với bàn phím (8 bytes chuẩn), **graphics tablet** có format riêng. Trong bài này:

```
Offset  Size   Ý nghĩa
──────────────────────────────────────────────────
[0]     1 byte  Button state:
                  0x00 = không nhấn (pen hover)
                  0x01 = left click (pen tip chạm bề mặt)
                  0x02 = right click (nút bên trên bút = SHIFT)
[1]     1 byte  Padding (luôn = 0x00)
[2:4]   2 bytes X coordinate (little-endian unsigned 16-bit)
[4:6]   2 bytes Y coordinate (little-endian unsigned 16-bit)
[6:8]   2 bytes Padding (luôn = 0x00 0x00)
```

**Ví dụ:**
```
Raw hex:  00 00 c6 1e cf 2c 00 00
           │  │  └──┘  └──┘  └──┘
           │  │  X=0x1EC6=7878  Y=0x2CCF=11471  padding
           │  padding
           btn=0 (hover)
```

### USB HID Keyboard Scancode

Bàn phím USB dùng **HID Usage ID** (scancode), KHÔNG phải ASCII. Bảng mapping quan trọng:

```
Scancode → Ký tự (không Shift) / Ký tự (có Shift)
──────────────────────────────────────────────────
0x04  →  a / A        0x1E  →  1 / !
0x05  →  b / B        0x1F  →  2 / @
0x06  →  c / C        0x20  →  3 / #
0x07  →  d / D        0x21  →  4 / $
0x08  →  e / E        0x22  →  5 / %
0x09  →  f / F        0x23  →  6 / ^
0x0A  →  g / G        0x24  →  7 / &
0x0B  →  h / H        0x25  →  8 / *
0x0C  →  i / I        0x26  →  9 / (
0x0D  →  j / J        0x27  →  0 / )
0x0E  →  k / K        0x2C  →  Space
0x0F  →  l / L        0x2D  →  - / _
0x10  →  m / M        0x2E  →  = / +
0x11  →  n / N        0x2F  →  [ / {
0x12  →  o / O        0x30  →  ] / }
0x13  →  p / P        0x33  →  ; / :
0x14  →  q / Q        0x34  →  ' / "
0x15  →  r / R        0x36  →  , / <
0x16  →  s / S        0x37  →  . / >
0x17  →  t / T        0x38  →  / / ?
0x18  →  u / U        0x28  →  Enter
0x19  →  v / V        0x2A  →  Backspace
0x1A  →  w / W        0x2B  →  Tab
0x1B  →  x / X        0x39  →  CapsLock
0x1C  →  y / Y
0x1D  →  z / Z
```

> **Tài liệu gốc:** [USB HID Usage Tables v1.12](https://usb.org/sites/default/files/documents/hut1_12v2.pdf) — Table 12, trang 53

---

### Decode dữ liệu ẩn

Sau khi biết gói tin là gì, tôi **export** với định dạng **csv** để phân tích:

**Output:**

```
"2717","17:42:00.171292","2.3.1","host","USB","72","0000d904860f0000","URB_INTERRUPT in"
"2719","17:42:00.175049","2.3.1","host","USB","72","0000c604860f0000","URB_INTERRUPT in"
"2721","17:42:00.262753","2.3.2","host","USB","72","0000000000000000","URB_INTERRUPT in"
"2723","17:42:00.262815","2.3.1","host","USB","72","0000c604860f0000","URB_INTERRUPT in"
...
```

Tôi viết script để **Decode** dữ liệu này:

```python
import csv
import struct

# Bảng tra cứu Scancode (HID Usage ID)
HID_NORMAL = {
    0x04:'a', 0x05:'b', 0x06:'c', 0x07:'d', 0x08:'e', 0x09:'f', 0x0a:'g', 0x0b:'h', 0x0c:'i', 0x0d:'j', 0x0e:'k', 0x0f:'l', 0x10:'m', 0x11:'n', 0x12:'o', 0x13:'p', 0x14:'q', 0x15:'r', 0x16:'s', 0x17:'t', 0x18:'u', 0x19:'v', 0x1a:'w', 0x1b:'x', 0x1c:'y', 0x1d:'z', 0x1e:'1', 0x1f:'2', 0x20:'3', 0x21:'4', 0x22:'5', 0x23:'6', 0x24:'7', 0x25:'8', 0x26:'9', 0x27:'0', 0x2c:' ', 0x2d:'-', 0x2e:'=', 0x2f:'[', 0x30:']'
}

HID_SHIFT = {
    0x04:'A', 0x05:'B', 0x06:'C', 0x07:'D', 0x08:'E', 0x09:'F', 0x0a:'G', 0x0b:'H', 0x0c:'I', 0x0d:'J', 0x0e:'K', 0x0f:'L', 0x10:'M', 0x11:'N', 0x12:'O', 0x13:'P', 0x14:'Q', 0x15:'R', 0x16:'S', 0x17:'T', 0x18:'U', 0x19:'V', 0x1a:'W', 0x1b:'X', 0x1c:'Y', 0x1d:'Z', 0x1e:'!', 0x1f:'@', 0x20:'#', 0x21:'$', 0x22:'%', 0x23:'^', 0x24:'&', 0x25:'*', 0x26:'(', 0x27:')', 0x2c:' ', 0x2d:'_', 0x2e:'+', 0x2f:'{', 0x30:'}'
}

def solve():
    flag = ""
    file_name = 'a.csv' # Đã đổi tên file sang a.csv

    try:
        # Mở file với chế độ đọc CSV
        with open(file_name, mode='r', encoding='utf-8') as f:
            # quotechar='"' giúp xử lý các nội dung nằm trong dấu ngoặc kép
            reader = csv.reader(f, delimiter=',', quotechar='"')
            
            print(f"[*] Đang quét file: {file_name}...")
            
            for row in reader:
                # Dựa trên ví dụ bạn đưa, hex data nằm ở cột thứ 7 (index 6)
                if len(row) < 7:
                    continue
                
                capdata = row[6].strip()
                
                # Kiểm tra độ dài payload (8 bytes = 16 ký tự hex)
                if len(capdata) == 16:
                    try:
                        b = bytes.fromhex(capdata)
                        
                        btn = b[0] # Byte 0: Trạng thái nút/Shift
                        # Byte 2-3: Tọa độ X (Little-endian)
                        x_val = struct.unpack('<H', b[2:4])[0] 
                        # Byte 4-5: Tọa độ Y (Little-endian)
                        y_val = struct.unpack('<H', b[4:6])[0]
                        
                        # Điều kiện lọc Flag ẩn (Y=0 và X là scancode)
                        if y_val == 0 and 0 < x_val < 100:
                            is_shift = (btn == 2)
                            table = HID_SHIFT if is_shift else HID_NORMAL
                            
                            char = table.get(x_val, f'[{hex(x_val)}]')
                            flag += char
                            print(f"  [+] Packet {row[0]}: Found '{char}'")
                            
                    except ValueError:
                        # Bỏ qua nếu dòng đó không phải hex hợp lệ
                        continue

        print("\n" + "="*40)
        print(f" KẾT QUẢ FLAG: {flag}")
        print("="*40)

    except FileNotFoundError:
        print(f"[!] Lỗi: Không tìm thấy file '{file_name}'.")

if __name__ == "__main__":
    solve()

```

**Output:**

```
[*] Đang quét file: a.csv...
  [+] Packet 1715: Found 'U'
  [+] Packet 1719: Found 'D'
  [+] Packet 1723: Found 'C'
  [+] Packet 1727: Found 'T'
  [+] Packet 1731: Found 'F'
  [+] Packet 1735: Found '{'
  [+] Packet 1741: Found 'k'
  [+] Packet 1745: Found '3'
  [+] Packet 1749: Found 'y'
  [+] Packet 1755: Found '_'
  [+] Packet 1759: Found 'S'
  [+] Packet 1765: Found 't'
  [+] Packet 1771: Found 'R'
  [+] Packet 1777: Found '0'
  [+] Packet 1783: Found 'K'
  [+] Packet 1789: Found '3'
  [+] Packet 1795: Found 'E'
  [+] Packet 1803: Found '_'
  [+] Packet 1813: Found '1'
  [+] Packet 1819: Found 'S'
  [+] Packet 1827: Found '_'
  [+] Packet 1837: Found '7'
  [+] Packet 1841: Found 'h'
  [+] Packet 1845: Found 'e'
  [+] Packet 1851: Found '_'
  [+] Packet 1857: Found 'w'
  [+] Packet 1863: Found 'A'
  [+] Packet 1869: Found 'y'
  [+] Packet 1875: Found '}'

========================================
 KẾT QUẢ FLAG: UDCTF{k3y_StR0K3E_1S_7he_wAy}
========================================
```

#### Bảng decode chi tiết

| Index | btn | X (hex) | X (dec) | Ký tự |
|-------|-----|---------|---------|-------|
| 1 | 2 (SHIFT) | 0x18 | 24 | **U** |
| 2 | 2 (SHIFT) | 0x07 | 7  | **D** |
| 3 | 2 (SHIFT) | 0x06 | 6  | **C** |
| 4 | 2 (SHIFT) | 0x17 | 23 | **T** |
| 5 | 2 (SHIFT) | 0x09 | 9  | **F** |
| 6 | 2 (SHIFT) | 0x2F | 47 | **{** |
| 7 | 0 (normal)| 0x0E | 14 | **k** |
| 8 | 0 (normal)| 0x20 | 32 | **3** |
| 9 | 0 (normal)| 0x1C | 28 | **y** |
| 10| 2 (SHIFT) | 0x2D | 45 | **_** |
| 11| 2 (SHIFT) | 0x16 | 22 | **S** |
| 12| 0 (normal)| 0x17 | 23 | **t** |
| 13| 2 (SHIFT) | 0x15 | 21 | **R** |
| 14| 0 (normal)| 0x27 | 39 | **0** |
| 15| 2 (SHIFT) | 0x0E | 14 | **K** |
| 16| 0 (normal)| 0x20 | 32 | **3** |
| 17| 2 (SHIFT) | 0x08 | 8  | **E** |
| 18| 2 (SHIFT) | 0x2D | 45 | **_** |
| 19| 0 (normal)| 0x1E | 30 | **1** |
| 20| 2 (SHIFT) | 0x16 | 22 | **S** |
| 21| 2 (SHIFT) | 0x2D | 45 | **_** |
| 22| 0 (normal)| 0x24 | 36 | **7** |
| 23| 0 (normal)| 0x0B | 11 | **h** |
| 24| 0 (normal)| 0x08 | 8  | **e** |
| 25| 2 (SHIFT) | 0x2D | 45 | **_** |
| 26| 0 (normal)| 0x1A | 26 | **w** |
| 27| 2 (SHIFT) | 0x04 | 4  | **A** |
| 28| 0 (normal)| 0x1C | 28 | **y** |
| 29| 2 (SHIFT) | 0x30 | 48 | **}** |

