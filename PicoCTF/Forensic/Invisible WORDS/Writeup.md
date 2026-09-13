# Write up Invisible WORDS

<img width="828" height="766" alt="image" src="https://github.com/user-attachments/assets/ffda0a72-9ae5-45e8-a007-6283f7a3a80d" />


**Mô tả**: Bạn có nhận ra nhân vật phản diện cyberpunk này không? Chúng tôi cũng không. Ngày nay, các công cụ tạo nghệ thuật AI đang là cơn thịnh nộ, điều này khiến cho việc có được một ảnh bìa đáng tin cậy trở nên khó khăn. Nhưng chúng tôi biết bạn sẽ tìm ra nó. Nghi phạm được cho là đang buôn bán kinh điển. Điều đó có lẽ sẽ không giúp phá được tấm bia, nhưng chúng tôi hy vọng nó sẽ tạo động lực để đưa tên tội phạm này ra trước công lý!

Để bài cho 1 file ảnh **ouput.bmp** nhìn lạ lạ và 2 **hint:**
- Something doesn't quite add up with this image...
- How's the image quality?

<img width="960" height="540" alt="image" src="https://github.com/user-attachments/assets/45af36f3-f174-404c-9c18-9536f758ac89" />

Kiểm tra file:
```
└─$ file output.bmp 
output.bmp: PC bitmap, Windows 98/2000 and newer format, 960 x 540 x 32, cbSize 2073738, bits offset 138
```

- `960 x 540`: chiều rộng x chiều cao của ảnh (số lượng pixel)
- ` x 32`: độ sau màu: 32 bit/pixel = 4 byte/pixel
- `cbSize 2073738`: tổng kích thước file thật sự trên đĩa
- `bits offset 138`: vị trí byte bắt đầu của vùng dữ liệu pixel (bỏ qua phần header BMP)
> Một ảnh 32-bit BMP thì: `1 pixel = 32 bit = 4 byte`,
ví dụ: `Pixel 1:  [B][G][R][A]`

Trong BMP, mấy byte đầu là header. Một field trong header tên là `bfOffBits`, nằm ở offset `0x0A` và dài 4 byte, field cho biết Pixel bắt đầu từ byte số bao nhiêu trong file.
Dùng công cụ ImHex ta biết được pixel data bắt đầu từ byte 138 trở đi.

<img width="767" height="575" alt="image" src="https://github.com/user-attachments/assets/848ac1a4-a0a2-4a63-8693-4534b4dc67ad" />


Nhìn vào ảnh ta còn thấy chữ `PK` cho biết file ảnh này đang giấu 1 file `ZIP`, header file Zip `​50 4B 03 04` nhưng ta thấy trong ảnh lại là `​50 4B 95 52 03 04` có vẻ 2 byte `95 52` bị chèn vào giữa.

Các magic byte khác: 
```
ZIP  50 4B 03 04
PNG  89 50 4E 47
JPG  FF D8 FF
PDF  25 50 44 46
ELF  7F 45 4C 46
```

Đây là khoảnh khắc xác nhận giả thuyết ban đầu: đúng như tên bài "Invisible **WORD**s" gợi ý, mỗi pixel 4-byte (1 DWORD) bị tách làm 2 WORD 2-byte — **WORD đầu giữ dữ liệu ảnh gốc** (để ảnh vẫn hiển thị bình thường, chỉ hơi nhiễu), còn **WORD sau bị ghi đè bởi dữ liệu ZIP ẩn**.

Dùng code python để bỏ 2 byte và giữ lại 2 byte và ghi ra file `output.zip`
```python
g = open("output.zip", "wb")

with open("output.bmp", "rb") as f:
    f.read(0x8a)

    skip = f.read(2)

    while skip:
        keep = f.read(2)
        g.write(keep)
        skip = f.read(2)

g.close()
```

Kiểm tra lại `file ouput.zip` thấy đã đúng header `50 4B 03 04`: 
```text
00000000   50 4B 03 04  14 00 00 00  08 00 68 13  70 56 77 A0  E6 54 AF 95  02 00 82 D8  06 00 1C 00  1C 00 5A 6E  4A 68 62 6D  74 6C 62 6E  4E 30 5A 57  6C 75 4C 58  52 6C 63 33  PK........h.pVw..T............ZnJhbmtlbnN0ZWluLXRlc3
00000034   51 75 64 48  68 30 55 54  09 00 03 83  7E 12 64 83  7E 12 64 75  78 0B 00 01  04 00 00 00  00 04 00 00  00 00 8C FD  CD 8E 24 E9  91 25 0A EE  13 C8 77 B0  C0 6C AA 1A  QudHh0UT....~.d.~.dux.................$..%....w..l..

```

Xác nhận có file 
zip bên trong:
```text
binwalk output.zip
output.zip: Zip archive data, at least v2.0 to extract, compression method=deflate

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             Zip archive data, at least v2.0 to extract, compressed size: 169391, uncompressed size: 448642, name: ZnJhbmtlbnN0ZWluLXRlc3QudHh0
169575        0x29667         End of Zip archive, footer length: 22
```

Giải nén và trích xuất file Zip ta được file `ZnJhbmtlbnN0ZWluLXRlc3QudHh0
` decode base64 thì có tên `frankenstein-test.txt`, đọc nó xem có gì và phát hiện flag.

<img width="935" height="135" alt="image" src="https://github.com/user-attachments/assets/72265765-8166-4d8b-af53-fe6f697845c7" />

## Flag
picoCTF{w0rd_d4wg_y0u_f0und_5h3113ys_m4573rp13c3_b48ea7de}
