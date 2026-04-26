# Writeup Dear Diary

---

<img width="944" height="726" alt="image" src="https://github.com/user-attachments/assets/ec7c0640-fc64-4fe2-91ad-a56c7eb64c1e" />

---

```
If you can find the flag on this disk image, we can close the case for good!
Download the disk image here.
```

**File đính kèm:** `disk.flag.img.gz`


```bash
gunzip disk.flag.img.gz
```

**`gunzip`** – giải nén file `.gz`. File disk image thường được nén để tiết kiệm bandwidth khi download.

Kiểm tra file vừa giải nén:

```bash
file disk.flag.img
```

**Output mong đợi:**
```
disk.flag.img: DOS/MBR boot sector; partition 1 : ID=0x83, active, start-CHS (0x0,32,33),
end-CHS (0x26,94,56), startsector 2048, 614400 sectors; partition 2 : ID=0x82, start-CHS
(0x26,94,57), end-CHS (0x47,1,58), startsector 616448, 524288 sectors; partition 3 : ID=0x83,
start-CHS (0x47,1,59), end-CHS (0x82,138,8), startsector 1140736, 956416 sectors
```

**output:**

| Partition | Type | Start Sector | Ý nghĩa |
|-----------|------|-------------|---------|
| 1 | 0x83 (ext4) | 2048 | Linux filesystem (boot) |
| 2 | 0x82 (swap) | 616448 | Linux swap |
| 3 | 0x83 (ext4) | **1140736** | Linux filesystem (DATA) |

---

## Xem cấu trúc partition (với mmls)

```bash
mmls disk.flag.img
```


**Output:**
```
DOS Partition Table
Offset Sector: 0
Units are in 512-byte sectors

      Slot      Start        End          Length       Description
000:  Meta      0000000000   0000000000   0000000001   Primary Table (#0)
001:  -------   0000000001   0000002047   0000002047   Unallocated
002:  000:000   0000002048   0000616447   0000614400   Linux (0x83)
003:  000:001   0000616448   0001140735   0000524288   Linux Swap / Solaris x86 (0x82)
004:  000:002   0001140736   0002097151   0000956416   Linux (0x83)
```

---


## Liệt kê các thư mục trong đĩa

```
sudo fls -o 1140736 disk.flag.img
d/d 32513:      home
d/d 11: lost+found
d/d 32385:      boot
d/d 64769:      etc
d/d 32386:      proc
d/d 13: dev
d/d 32387:      tmp
d/d 14: lib
d/d 32388:      var
d/d 21: usr
d/d 32393:      bin
d/d 32395:      sbin
d/d 32539:      media
d/d 203:        mnt
d/d 32543:      opt
d/d 204:        root
d/d 32544:      run
d/d 205:        srv
d/d 32545:      sys
d/d 32530:      swap
V/V 119417:     $OrphanFiles
```

Đọc thư mục **root**

```
sudo fls -o 1140736 disk.flag.img 204
r/r 1837:       .ash_history
d/d 1842:       secret-secrets
```

Phát hiện thư mục **secret-secrets** gấy chú ý, đọc nó và thấy có vẻ như hint !

```
sudo fls -o 1140736 disk.flag.img 1842
r/r 1843:       force-wait.sh
r/r 1844:       innocuous-file.txt
r/r 1845:       its-all-in-the-name
```


## Đọc raw directory data với icat

```bash
icat -o 1140736 disk.flag.img 8 | xxd | grep -A3 "\.txt"
```

**`icat`** – đọc nội dung của một inode cụ thể. Inode 8 thường là **root directory** trong ext4.

| Lệnh | Ý nghĩa |
|------|---------|
| `icat -o 1140736 disk.flag.img 8` | Đọc raw data của inode 8 (root dir) |
| `xxd` | Chuyển binary sang hex dump để dễ đọc |
| `grep -A3 ".txt"` | Tìm dòng có `.txt` và 3 dòng tiếp theo |
| `8`| Là Inode của Journal. Bản chất Journal là một "cuốn băng ghi hình" liên tục các thao tác ghi đĩa. Mọi thứ bạn viết vào file .txt đều phải đi qua đây trước |

**Output (trích):**
```
00004010: 3400 0000 1c00 1201 696e 6e6f 6375 6f75  4.......innocuou
00004020: 732d 6669 6c65 2e74 7874 0000 3500 0000  s-file.txt..5...
00004030: a803 0301 6f43 5400 0000 0000 0000 0000  ....oCT.........
...                   
                        
```
Nhìn vào **output** thấy các phần của flag:

---

Người thiết kế challenge đã:

1. **Tạo file** tên `innocuous-file.txt`
2. **Rename file** nhiều lần với lần lượt các tên: `pic`, `oCT`, `F{1`, `_53`, `3_n`, `4m3`, `5_8`, `0d2`, `4b3`, `0}`
3. Cuối cùng **rename lại** thành `innocuous-file.txt`

Mỗi lần rename, ext4 tạo ra một directory entry mới với tên mới, nhưng entry cũ **vẫn còn lưu trong block** dưới dạng "deleted entry" (rec_len của entry trước bị kéo dài để bao phủ entry cũ).

## Giải bằng autopsy

<img width="1100" height="490" alt="image" src="https://github.com/user-attachments/assets/a09f9305-d1f5-4306-b630-4a7a5155a62d" />

Search **.txt** và tìm lần lượt các **inode** sẽ thấy các mảnh **flag** dưới mã HEX như trong ảnh.

## Flag: picoCTF{1_533_n4m35_80d24b30}
