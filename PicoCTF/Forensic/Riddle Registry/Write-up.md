# Write up Riddle Registry - picoCTF
**Loại:** Phân tích siêu dữ liệu (Metadata Analysis)

---

# 1.Mô tả

Bài cung cấp 1 file pdf với nội dung ...

---

Mở file thì không thấy gì nên tôi dùng **exiftool** để xem metadata file

---

# 2.Bước giải

Sử dụng lệnh: 

```bash
exiftool confidential.pdf
```

hoặc 

```bash
pdfinfo confidential.pdf
```
Output:

```
└─$ exiftool confidential.pdf               
ExifTool Version Number         : 13.25
File Name                       : confidential.pdf
Directory                       : .
File Size                       : 183 kB
File Modification Date/Time     : 2026:03:06 17:09:22+07:00
File Access Date/Time           : 2026:03:06 17:09:29+07:00
File Inode Change Date/Time     : 2026:03:06 17:09:22+07:00
File Permissions                : -rw-rw-r--
File Type                       : PDF
File Type Extension             : pdf
MIME Type                       : application/pdf
PDF Version                     : 1.7
Linearized                      : No
Page Count                      : 1
Producer                        : PyPDF2
Author                          : cGljb0NURntwdXp6bDNkX20zdGFkYXRhX2YwdW5kIV8zNTc4NzM5YX0=
```

---

- Thấy trường **Author** có giá trị là base64
- Decode base64 thu được flag: **picoCTF{puzzl3d_m3tadata_f0und!_3578739a}**




