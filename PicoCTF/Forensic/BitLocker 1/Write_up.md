# \[Write-up\] - Bitlocker-1

## 1. Thông tin

Nhận được một file ảnh đĩa (.dd, .img) và mô tả về việc mã hóa mật
khẩu

Jacky is not very knowledgable about the best security passwords and used a simple password to encrypt their BitLocker drive. See if you can break through the encryption!
Download the disk image [here](https://challenge-files.picoctf.net/c_verbal_sleep/9e934e4d78276b12e27224dac16e50e6bbeae810367732eee4d5e38e6b2bb868/bitlocker-1.dd)

Hint: Hash cracking
<img width="959" height="617" alt="image" src="https://github.com/user-attachments/assets/099a373d-8979-4e72-849a-f9e09373b3bd" />

------------------------------------------------------------------------

## 2. Các công cụ

### A. bitlocker2john

-   Script thuộc John the Ripper.
-   Dùng để trích xuất metadata BitLocker sang dạng hash.


### B. John the Ripper

-   Công cụ bẻ khóa mật khẩu phổ biến.


### C. Hashcat

-   Công cụ crack mật khẩu


### D. Dislocker

-   Tool đọc phân vùng BitLocker trên Linux và giải mã ổ đĩa bị mã hóa bởi BitLoker (Windows).


------------------------------------------------------------------------

## 3. Các bước giải

### Bước 1: Kiểm tra file

``` bash
file bitlocker-1.dd
```

Kết quả:

```
└─$ file bitlocker-1.dd 
bitlocker-1.dd: DOS/MBR boot sector, code offset 0x58+2, OEM-ID "-FVE-FS-", sectors/cluster 8, reserved sectors 0, Media descriptor 0xf8, sectors/track 63, heads 255, hidden sectors 124499968, FAT (32 bit), sectors/FAT 8160, serial number 0, unlabeled; NTFS, sectors/track 63, physical drive 0x1fe0, $MFT start cluster 393217, serial number 02020454d414e204f, checksum 0x41462020
```

... OEM-ID "-FVE-FS-

→ Xác nhận BitLocker.

------------------------------------------------------------------------

### Bước 2: Trích xuất Hash

``` bash
bitlocker2john -i bitlocker-1.dd > hash.txt
```

Kết quả:

```
└─$ cat hash.txt 
Encrypted device bitlocker-1.dd opened, size 100MB
Salt: 2b71884a0ef66f0b9de049a82a39d15b
RP Nonce: 00be8a46ead6da0106000000
RP MAC: a28f1a60db3e3fe4049a821c3aea5e4b
RP VMK: a1957baea68cd29488c0f3f6efcd4689e43f8ba3120a33048b2ef2c9702e298e4c260743126ec8bd29bc6d58

UP Nonce: d04d9c58eed6da010a000000
UP MAC: 68156e51e53f0a01c076a32ba2b2999a
UP VMK: fffce8530fbe5d84b4c19ac71f6c79375b87d40c2d871ed2b7b5559d71ba31b6779c6f41412fd6869442d66d


User Password hash:
$bitlocker$0$16$cb4809fe9628471a411f8380e0f668db$1048576$12$d04d9c58eed6da010a000000$60$68156e51e53f0a01c076a32ba2b2999afffce8530fbe5d84b4c19ac71f6c79375b87d40c2d871ed2b7b5559d71ba31b6779c6f41412fd6869442d66d
Hash type: User Password with MAC verification (slower solution, no false positives)
$bitlocker$1$16$cb4809fe9628471a411f8380e0f668db$1048576$12$d04d9c58eed6da010a000000$60$68156e51e53f0a01c076a32ba2b2999afffce8530fbe5d84b4c19ac71f6c79375b87d40c2d871ed2b7b5559d71ba31b6779c6f41412fd6869442d66d
Hash type: Recovery Password fast attack
$bitlocker$2$16$2b71884a0ef66f0b9de049a82a39d15b$1048576$12$00be8a46ead6da0106000000$60$a28f1a60db3e3fe4049a821c3aea5e4ba1957baea68cd29488c0f3f6efcd4689e43f8ba3120a33048b2ef2c9702e298e4c260743126ec8bd29bc6d58
Hash type: Recovery Password with MAC verification (slower solution, no false positives)
$bitlocker$3$16$2b71884a0ef66f0b9de049a82a39d15b$1048576$12$00be8a46ead6da0106000000$60$a28f1a60db3e3fe4049a821c3aea5e4ba1957baea68cd29488c0f3f6efcd4689e43f8ba3120a33048b2ef2c9702e298e4c260743126ec8bd29bc6d58
```
------------------------------------------------------------------------

### Bước 3: Bẻ khóa mật khẩu

Cho dòng này vào file **hashuser.txt** để crack

```
$bitlocker$0$16$cb4809fe9628471a411f8380e0f668db$1048576$12$d04d9c58eed6da010a000000$60$68156e51e53f0a01c076a32ba2b2999afffce8530fbe5d84b4c19ac71f6c79375b87d40c2d871ed2b7b5559d71ba31b6779c6f41412fd6869442d66d
```

#### Cách 1: John

``` bash
john --wordlist=/usr/share/wordlists/rockyou.txt hashuser.txt
```

#### Cách 2: Hashcat

``` bash
hashcat -m 22100 hash_user.txt /usr/share/wordlists/rockyou.txt
```

Kết quả:

```
$bitlocker$1$16$cb4809fe9628471a411f8380e0f668db$1048576$12$d04d9c58eed6da010a000000$60$68156e51e53f0a01c076a32ba2b2999afffce8530fbe5d84b4c19ac71f6c79375b87d40c2d871ed2b7b5559d71ba31b6779c6f41412fd6869442d66d:jacqueline
```                                                          

**Password tìm được:** `jacqueline`

------------------------------------------------------------------------

### Bước 4: Giải mã và Mount

``` bash
sudo mkdir /mnt/decrypt
sudo dislocker -V bitlocker-1.dd -ujacqueline -- /mnt/decrypt
```
- **-V bitlocker-1.dd**: file ổ đĩa (image) đã bị mã hóa.
- **-u jacqueline**: pass của BitLocker đã crack đc ở trên.
- **-- /mnt/decrypt**: output sau khi decrypt sẽ nằm ở đây.

Sau lệnh này trong **/mnt/decrypt** xẽ xuất hiện file:
```
dislocker-file
```
> Đây là ổ đĩa đã được giải mã nhưng chưa mount

Tạo chỗ mount filesystem:

``` bash
sudo mkdir /mnt/flag
sudo mount -o loop /mnt/decrypt/dislocker-file /mnt/flag
```

- **mount**: gắn ổ đĩa vào hệ thống.
- **-o loop**: mount file như 1 ổ đĩa thật.
- **/mnt/decrypt/dislocker-file**: ổ đã decrpyt.
- **/mnt/flag**: nơi chứa data.


------------------------------------------------------------------------

### Bước 5: Lấy Flag

``` bash
ls /mnt/flag
cat /mnt/flag/flag.txt
```

```
└─$ # Liệt kê tất cả các file có trong ổ đĩa
ls -R /mnt/flag_content

# Tìm và đọc file flag (thường là flag.txt)
cat /mnt/flag_content/flag.txt
/mnt/flag_content:
'$RECYCLE.BIN'   flag.txt  'System Volume Information'

'/mnt/flag_content/$RECYCLE.BIN':
S-1-5-21-3576963320-1344788273-4164204335-1001

'/mnt/flag_content/$RECYCLE.BIN/S-1-5-21-3576963320-1344788273-4164204335-1001':
desktop.ini

'/mnt/flag_content/System Volume Information':
FVE2.{24e6f0ae-6a00-4f73-984b-75ce9942852d}    FVE2.{e40ad34d-dae9-4bc7-95bd-b16218c10f72}.2
FVE2.{aff97bac-a69b-45da-aba1-2cfbce434750}.1  FVE2.{e40ad34d-dae9-4bc7-95bd-b16218c10f72}.3
FVE2.{aff97bac-a69b-45da-aba1-2cfbce434750}.2  IndexerVolumeGuid
FVE2.{da392a22-cae0-4f0f-9a30-b8830385d046}    WPSettings.dat
FVE2.{e40ad34d-dae9-4bc7-95bd-b16218c10f72}.1
picoCTF{us3_b3tt3r_p4ssw0rd5_pl5!_3242adb1}  
```

**Flag:**

    picoCTF{us3_b3tt3r_p4ssw0rd5_pl5!_3242adb1}
