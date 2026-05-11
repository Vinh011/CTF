# Write up: Very Very Very Hidden

<img width="840" height="670" alt="image" src="https://github.com/user-attachments/assets/c57da0d2-063e-4ba5-a356-8e26bc1f2dc6" />

---

- Đề bài cho file **pcap**, sau khi lục lọi và lọc traffic **http** thì phát hiện có tải vài file ảnh nên export là xem có gì.

<img width="1919" height="1019" alt="image" src="https://github.com/user-attachments/assets/12b9984b-d44d-4110-bd94-54d6fa0cbac4" />

- Có 2 file ảnh, nhưng file ảnh lại khác nhau về kích thước và chật lượng, có vẻ có ẩn dấu gì bên trong đó, ngó lại với **hint** bài thì có nhắc đến truy vẫn nên tôi dùng filter xem có gì.  

```
-rw-r--r-- 1 zing zing 1284036 May 11 15:08 duck.png
-rw-r--r-- 1 zing zing 2497784 May 11 15:08 evil_duck.png
```

<img width="1919" height="1018" alt="image" src="https://github.com/user-attachments/assets/e91ea438-6e59-4920-9799-d8ea85faed9b" />

Thấy các **domain**:
- google.com
- github.com
- powershell.org
- teams.mirosoft.com

Có vẻ như hacker tìm kiếm tool gì đó trên github. Lục lọi 1 lúc phát hiện github [này](https://github.com/peewpw/Invoke-PSImage) nó là kĩ thuật Invoke-PSImage mã hóa một PowerShell script vào trong các pixel của một file PNG. 

Repo dùng `kiwi.png` → output `evil_kiwi.png`. Bài CTF dùng `duck.png` → `evil_duck.png`. Nhờ AI vibe code script để extract PowerShell script ẩn.

```
└─$ python3 decode_psimage.py 
[+] Done!
[+] Preview:
$out = "flag.txt"
$enc = [system.Text.Encoding]::UTF8
$string1 = "HEYWherE(IS_tNE)50uP?^DId_YOu(]E@t*mY_3RD()B2g3l?"
$string2 = "8,:8+14>Fx0l+$*KjVD>[o*.;+1|*[n&2G^201l&,Mv+_'T_B"

$data1 = $enc.GetBytes($string1)
$bytes = $enc.GetBytes($string2)

for($i=0; $i -lt $bytes.count ; $i++)
{
```

Tìm đc **key** và **ciphertext**, đem đi xor ở **cyberchef**.


<img width="1913" height="912" alt="image" src="https://github.com/user-attachments/assets/cee46b5e-5c50-4f56-ba1d-93148c3ca8bf" />

### Flag: picoCTF{n1c3_job_f1nd1ng_th3_s3cr3t_in_the_im@g3}














