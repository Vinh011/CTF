# Write up Hidden in plainsight — PicoCTF
**Loại:** Ẩn tin (Steganography)
          Phân tích chữ ký tệp & Hex (File Signature & Hex Analysis)

---

# 1. Mô tả

Gợi ý challenge:

> "Something is tucked away out sight inside thi file."

Ý nghĩa của câu này là: 

- Có dữ liệu được **giấu bên trong file**
- Nhưng nó **không hiển thị trực tiếp**

Chúng ta được cung cấp **một file ảnh JPG**

---

# 2. Bước giải

Tôi dùng **file** để check định dạng file:

Output:

```
$ file img.jpg  
img.jpg: JPEG image data, JFIF standard 1.01, aspect ratio, density 1x1, segment length 16, comment: "c3RlZ2hpZGU6Y0VGNmVuZHZjbVE9", baseline, precision 8, 640x640, components 3
```

---

Dùng exiftool để đọc metadata:

```
└─$ exiftool img.jpg 
ExifTool Version Number         : 13.25
File Name                       : img.jpg
Directory                       : .
File Size                       : 73 kB
File Modification Date/Time     : 2026:03:06 17:45:25+07:00
File Access Date/Time           : 2026:03:06 17:45:38+07:00
File Inode Change Date/Time     : 2026:03:06 17:45:28+07:00
File Permissions                : -rw-rw-r--
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
JFIF Version                    : 1.01
Resolution Unit                 : None
X Resolution                    : 1
Y Resolution                    : 1
Comment                         : c3RlZ2hpZGU6Y0VGNmVuZHZjbVE9
Image Width                     : 640
Image Height                    : 640
Encoding Process                : Baseline DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:2:0 (2 2)
Image Size                      : 640x640
Megapixels                      : 0.410
```

Trong kết quả ta thấy một trường đáng nghi:

```
Comment: c3RlZ2hpZGU6Y0VGNmVuZHZjbVE9
```

---

Decode Base64

```bash
echo "c3RlZ2hpZGU6Y0VGNmVuZHZjbVE9" | base64 -d
```

Output:

```
steghide:cEF6endvcmQ=
```

Điều này gợi ý rằng:
- Dữ liệu được giấu bằng **steghide**
- Phần sau dấu : là Base64 khác

---

Giải mã chuỗi thứ hai:

```
echo "cEF6endvcmQ=" | base64 -d
```

Output:

```
pAzzword
```

--- 

Kiểm tra xem ảnh có dữ liệu ẩn không:
- Sử dụng tool **steghide** để xem thông tin ẩn với password là: **pAzzword**

```
steghide info img.jpg
```

Output:

```
└─$ steghide info img.jpg
"img.jpg":
  format: jpeg
  capacity: 4.0 KB
Try to get information about embedded data ? (y/n) y
Enter passphrase: 
  embedded file "flag.txt":
    size: 34.0 Byte
    encrypted: rijndael-128, cbc
    compressed: yes

```

File có dữ liệu ẩn:

```
  embedded file "flag.txt":
    size: 34.0 Byte
    encrypted: rijndael-128, cbc
    compressed: yes
```

---

Extract dữ liệu:

```bash
steghide extract -sf img.jpg -p pAzzword
```

Output:

```
wrote extracted data to "flag.txt"
```

Flag: **picoCTF{h1dd3n_1n_1m4g3_e7f5b969}**








