# Write up: bitlocker-2
<img width="954" height="639" alt="image" src="https://github.com/user-attachments/assets/63dd9f20-4faf-46b0-a575-6796f8d93581" />

## Mô tả
>Jacky has learnt about the importance of strong passwords and made sure to encrypt the BitLocker drive with a very long and complex password. We managed to capture the RAM while this drive was opened however. See if you can break through the encryption!

**File cho trước:**
- `bitlocker-2.dd`: Disk image được mã hóa bằng BitLocker, [tải](https://challenge-files.picoctf.net/c_verbal_sleep/b22e1ca13c0b82bb85afe5ae162f6ecbdf5b651e364e6a2b57c9ad44ae0b3bfd/bitlocker-2.dd)
- `memdum.mem.gz`: RAM dump chụp lại khi drive đang được mở, [tải](https://challenge-files.picoctf.net/c_verbal_sleep/b22e1ca13c0b82bb85afe5ae162f6ecbdf5b651e364e6a2b57c9ad44ae0b3bfd/memdump.mem.gz)

---

## Ý đồ bài 

Bài này kiểm tra khả năng **Memory Forensic** kết hợp với **Disk Decryption**.

Khi BitLocker đang hoạt động và drive đang mở (unlocked), Windows phải giữ **FVEK (Full Volume Encryption Key)** trong RAM để liên tục mã hóa/giải mã dữ liệu đọc ghi. Đây là điểm mẫu chốt:

> **Nếu chụp được RAM đúng lúc drive đang mở -> có thể trích xuất FVEK từ RAM -> giải mã toàn bộ disk mà không cần password gốc.**

Mật khẩu BitLocker dù dài và phức tạp đến đâu cũng không an toàn nữa, vì ta pybass hoàn toàn bằng key đang nằm sẵn trong bộ nhớ.

---

## Hướng giải 

```
memdump.mem (RAM dump)
        │
        ▼
  Volatility 2 + BitLocker plugin
  → Scan memory pool tìm FVEK
        │
        ▼
  File .fvek (FVEK key đã dump)
        │
        ▼
  Dislocker giải mã bitlocker-2.dd
  → Tạo virtual disk đã decrypt
        │
        ▼
  Mount NTFS bằng ntfs-3g
        │
        ▼
  Đọc flag.txt
```

**Tại sao FVEK nằm trong RAM?**

BitLocker dùng kiến trúc phân tầng:
- **Password** -> giải mã  **VMK (Volume Master Key)**
- **VMK** -> giải mã -> **FVEK (Full Volume Encryption Key)**
- **FVEK** -> dùng trực tiếp để mã hóa/giải mã data trên disk

---

## Tool sử dụng 

| Tool | Mục đích |
|------|----------|
| `volatility2` | Phân tích RAM dump, trích xuất FVEK |
| `BitLocker plugin (breppo)` | Plugin scan memory pool tìm FVEK | 
| `dislocker` | Giải mã BitLocker disk image bằng FVEK file |
| `ntfs-3g` | Mount filesystem NTFS từ disk đã giải mã | 

--- 

## Các bước giải

### Bước 1: Giải nén RAM dump

```bash
gunzip memdump.mem.gz
```

File sau khi giải nén: `memdump.mem` (~2GB)


--- 

### Bước 2: Cài Volatility 2 nếu chưa có

```bash
git clone https://github.com/volatilityfoundation/volatility.git vol2
cd vol2
pip2 install pycryptodome distorm3
```

> Bài này dùng **Volatility 2** thay vì **Volatility 3** vì Plugin BitLocker được viết cho API của Vol2.

--- 


### Bước 3: Cài BitLocker plugin cho Volatility 2

Volatility 2 mặc định không có plugin BitLocker, cần cài thêm thủ công:

> plugin là module mở rộng dành cho tool Volatility, dùng để trích xuất key mã hóa BitLocker (FVEK) từ RAM dump.

Kiểm tra plugin đẫ được nhận:

```bash
python2 vol2/vol.py --help | grep bitlocker
```

---

### Bước 4: Xác định profile 

```bash
python2 vol2/vol.py -f memdump.mem imageinfo
```

- `-f memdump.mem`: Chỉ định file RAM dump đầu vào
- `imageinfo`: Plugin tự động nhận diện OS profile bằng cách tìm cấu trúc KDBG (Kerenl Debugger Block) trong memory

**Output:**

```
Suggested Profile(s) : Win10x64_19041
Image date and time  : 2025-03-10 02:58:56 UTC+0000
Number of Processors : 2
```

Profile `Win10x64_19041` tương đương Windows 10 Build 19041 (version 20H1).

---

### Bước 5: Trích xuất FVEK từ RAM dump

- Tạo thư mục để dump key

```bash
mkdir -p dumps/
```

```bash
python2 vol2/vol.py -f memdump.mem --profile=Win10x64_19041 bitlocker --dislocker dumps/
```

- `--profile=Win10x64_19041`: Chỉ định profile OS để Volatility parse đúng cấu trúc kerenl như đã tìm được ở trên
- `bitlocker`: Gọi BitLocker plugin, scan các memory pool tag `FVEc` và `Cngb` để tìm FVEK
- `--dislocker dumps/`: Dump key ra thư mục theo định dạng binary tương thích với tool `dislocker` (khác với `--dump-dir` chỉ lưu dạng text)

**Output:**
```
[FVEK] Address : 0x8087865bead0
[FVEK] Cipher  : AES-XTS 128 bit (Win 10+)
[FVEK] FVEK: 4f79d4a00d5e9b25965b89581a6a599c
[DUMP] FVEK dumped to file: dumps/0x8087865bead0-Dislocker.fvek

[FVEK] Address : 0x40d857c90
[FVEK] Cipher  : AES 128-bit (Win 8+)
[FVEK] FVEK: d40582190eb6f067691120bbbe55e511
...
```

Plugin tìm thấy nhiều FVEK vì Windows giữ nhiều bản copy trong các vùng memory khác nhau.

**Key dúng là `0x8087865bead0-Dislocker.fvek`** vì: 
- Cipher **AES-XTS 128-bit** là cipher mặc định của Windows 10+
- Khớp với OS profile `Win10x64_1904` như đã tìm được trước đó

---

### Bước 6: Tạo thư mục mout

```bash
sudo mkdir -p /mnt/dislocker
sudo mkdir -p /mnt/decryted
```

--- 

### Bước 7: Giải mã disk bằng dislocker

```bash
sudo dislocker -V bitlocker-2.dd -k dumps/0x8087865bead0-Dislocker.fvek --/mnt/dislocker
```

**Giải thích option:**
- `-V bitlocker-2.dd`: Chỉ định file disk image BitLocker cần giải mã
- `-k dumps/0x8087865bead0-Dislocker.fvek`: Dùng **file FVEK binary** đã dump ở bước 5
- `-- /mnt/dislocker`: Mount point cho virtual disk sau khi decrypt

> **Quang trọng:** Phải dùng flag `-k` với **đường dẫn file** `.fvek`, **không phải** `--fvek 0x...` với hex string. Flag `--fvek` của dislocker yêu cầu format khác và sẽ fail với các key này.

Lệnh này tạo ra file `/mnt/dislocker/dislocker-file1`: đây là virtual disk đã giải mã hoàn toàn, sẵn sàng để mount như filesystem bình thường.

---

### Bước 8: Mount filesystem NTFS
```bash
sudo mount -o loop /mnt/dislocker/dislocker-file /mnt/decrypted
```

Nếu sảy ra lỗi: 

```
The disk contains an unclean file system (0, 0).
Metadata kept in Windows cache, refused to mount.
Falling back to read-only mount...
unknown filesystem type 'ntfs'
```
> Thì thử lệnh bên dưới 

```bash
sudo mount -t ntfs-3g /mnt/dislocker/dislocker-file /mnt/decrypted
```

**Option:** 
- `-t ntfs-3g`: Bắt buộc chỉ định loại filesystem NTFS, dùng driver `ntfs-3g`
- `/mnt/dislocker/dislocker-file`: Virual disk đã giải mã từ bước 7
- `/mnt/decrypted`: Thư mục để truy cập nội dung

> **Quan trọng:** Không dùng `-o loop` thông thường. Filesystem NTFS này ở trạng thái "unclean" (drive chưa được unmount sạch trước khi chụp RAM), nên cần `ntfs-3g` để mount ở chế độ read-only.

---

### Bước 9: Đọc flag

```bash
ls /mnt/decrypted
```

**Output**: 

```
# $RECYCLE.BIN  System Volume Information  flag.txt
```

Đọc flag:

```bash
cat /mnt/decrypted/flag.txt
```
**Flag:**
```
picoCTF{B1tl0ck3r_dr1v3_d3crypt3d_9029ae5b}
```

## Unintended Solution (cách tắt)

Ngoài cách intended ở trên, bài này có một shortcut rất đơn giản:

```bash
strings memdump.mem | grep "picoCTF"
```

**Output**: 

```
picoCTF{B1tl0ck3r_dr1v3_d3crypt3d_9029ae5b}
```

> Vì drive đang được mở tại thời điểm chụp RAM, flag.txt đã được đọc vào bộ nhớ dưới dạng plaintext. `strings` chỉ đơn giản scan toàn bộ byte trong file va in ra các chuỗi ký tự có thể đọc được - flag nằm sẵn ở đó.

- [Volatility-BitLocker plugin](https://github.com/breppo/Volatility-BitLocker)
- [Volatility Foundation Vol2](https://github.com/volatilityfoundation/volatility)
- [Dislocker](https://github.com/Aorimn/dislocker)


