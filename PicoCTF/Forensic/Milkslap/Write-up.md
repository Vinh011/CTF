# 🚩 Write-up: picoCTF - MilkSlap

## Thông tin

- **Thể loại:** Web / Forensics  
- **Mô tả:** Tìm flag ẩn trong một trang web.  
- **Hint:** Look at the problem category.
<img width="955" height="638" alt="image" src="https://github.com/user-attachments/assets/3ea9de4a-524f-4226-ab6e-9ecb8228b9de" />

---

### Kiểm tra source code

Mở DevTools (F12) → tab **Elements / Sources**

Tìm thấy một file ảnh đáng chú ý:

```
/static/concat_v.png
```

---

### Phân tích file ảnh

Tải ảnh về:

```bash
wget http://<target>/static/concat_v.png
```

Kiểm tra nhanh:

```bash
file concat_v.png
```

Kết quả:

```
PNG image data, 1280 x 47520
```

---

### Phân tích steganography bằng zsteg

```bash
zsteg concat_v.png
```
Do ảnh có chiều cao lớn nên xảy ra lỗi

```
└─$ zsteg -s all concat_v.png 
/var/lib/gems/3.3.0/gems/zpng-0.4.5/lib/zpng/scan_line.rb:369:in `prev_scanline_byte': stack level too deep (SystemStackError)
        from /var/lib/gems/3.3.0/gems/zpng-0.4.5/lib/zpng/scan_line.rb:319:in `block in decoded_bytes'
        from /var/lib/gems/3.3.0/gems/zpng-0.4.5/lib/zpng/scan_line.rb:318:in `upto'
        from /var/lib/gems/3.3.0/gems/zpng-0.4.5/lib/zpng/scan_line.rb:318:in `decoded_bytes'
        from /var/lib/gems/3.3.0/gems/zpng-0.4.5/lib/zpng/scan_line/mixins.rb:17:in `prev_scanline_byte'
        from /var/lib/gems/3.3.0/gems/zpng-0.4.5/lib/zpng/scan_line.rb:377:in `prev_scanline_byte'
        from /var/lib/gems/3.3.0/gems/zpng-0.4.5/lib/zpng/scan_line.rb:319:in `block in decoded_bytes'
        from /var/lib/gems/3.3.0/gems/zpng-0.4.5/lib/zpng/scan_line.rb:318:in `upto'
        from /var/lib/gems/3.3.0/gems/zpng-0.4.5/lib/zpng/scan_line.rb:318:in `decoded_bytes'
         ... 10225 levels...
        from /var/lib/gems/3.3.0/gems/zsteg-0.2.13/lib/zsteg.rb:26:in `run'
        from /var/lib/gems/3.3.0/gems/zsteg-0.2.13/bin/zsteg:8:in `<top (required)>'
        from /usr/local/bin/zsteg:25:in `load'
        from /usr/local/bin/zsteg:25:in `<main>'
```
Tôi dùng scrip python để cắt ảnh

```
from PIL import Image
import os

img = Image.open("concat_v.png")
print("Size:", img.size)

chunk_height = 3000
width, height = img.size

os.makedirs("chunks", exist_ok=True)

idx = 0
for top in range(0, height, chunk_height):
    bottom = min(top + chunk_height, height)
    chunk = img.crop((0, top, width, bottom))
    out = f"chunks/chunk_{idx:02d}.png"
    chunk.save(out)
    print("Saved", out, chunk.size)
    idx += 1 
```

---

Sau khi cắt thành các ảnh nhỏ hơn, tôi dùng lệnh bash để tự động chạy **zsteg**

```bash
└─$ for f in chunks/*.png; do
    echo "=== $f ==="
    zsteg "$f"
done
```
### kết quả

```
=== chunks/chunk_00.png ===
imagedata           .. text: "\n\n\n\n\n\n\t\t"
b1,b,lsb,xy         .. text: "picoCTF{imag3_m4n1pul4t10n_sl4p5}\n"
b2,r,lsb,xy         .. text: ["U" repeated 8 times]
b2,r,msb,xy         .. file: VISX image file
b2,g,lsb,xy         .. file: VISX image file
b2,g,msb,xy         .. file: SoftQuad DESC or font file binary - version 15722
b2,b,msb,xy         .. text: "UfUUUU@UUU"
b4,r,lsb,xy         .. text: "\"\"\"\"\"#4D"
b4,r,msb,xy         .. text: "wwww3333"
b4,g,lsb,xy         .. text: "wewwwwvUS"
```

---

# #Flag

```
picoCTF{imag3_m4n1pul4t10n_sl4p5}
```

---

