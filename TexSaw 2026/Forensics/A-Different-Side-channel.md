# \[Write-up\] A Different Side Channel

Forensic: Side-Channel Attack on AES

## 1. Tổng quan
<img width="623" height="728" alt="image" src="https://github.com/user-attachments/assets/ab1a54b2-6dab-4e50-977d-df94c37b4e79" />

**Đề bài:** Cung cấp các file đo lường năng lượng (`traces.npy`), dữ
liệu đầu vào (`plaintexts.npy`) và một file bị mã hóa
(`encrypted_flag.bin`).

**Mục tiêu:** Khôi phục khóa AES-128 bằng phương pháp tấn công kênh kề
(Side-channel Attack) và giải mã Flag.

------------------------------------------------------------------------

## 2. Phân tích và Tư duy

### Ý đồ bài toán

Bài toán mô phỏng một cuộc tấn công thực tế vào phần cứng (ví dụ: thẻ
thông minh hoặc chip nhúng).\
Dù thuật toán AES rất an toàn, nhưng khi chạy trên chip, nó tiêu thụ
điện năng khác nhau tùy thuộc vào dữ liệu đang xử lý.

Sự chênh lệch này bị rò rỉ ra ngoài và được ghi lại thành các "traces".

### Phương pháp: Correlation Power Analysis (CPA)

-   Tấn công vào vòng 1 của AES (sau SubBytes)
-   Mô hình rò rỉ: **Hamming Weight (HW)**

### Tư duy giải

-   Với mỗi byte khóa (256 khả năng)
-   Tính HW lý thuyết
-   So sánh với trace bằng hệ số tương quan Pearson
-   Key có tương quan cao nhất → key đúng

------------------------------------------------------------------------

## 3. Giai đoạn 1: Tìm khóa

File: `find_key.py`

``` python
import numpy as np

SBOX = np.array([...], dtype=np.uint8)
HW = np.array([bin(i).count('1') for i in range(256)], dtype=np.uint8)

traces = np.load('traces.npy')
plaintexts = np.load('plaintexts.npy')
traces_diff = traces - np.mean(traces, axis=0)

best_key = []

for b in range(16):
    max_c = -1
    found_byte = 0
    for k in range(256):
        hyp_hw = HW[SBOX[plaintexts[:, b] ^ k]].astype(float)
        hyp_hw_diff = hyp_hw - np.mean(hyp_hw)

        num = np.dot(hyp_hw_diff, traces_diff)
        den = np.sqrt(np.sum(hyp_hw_diff**2) * np.sum(traces_diff**2, axis=0))
        corr = np.max(np.abs(np.nan_to_num(num / den)))

        if corr > max_c:
            max_c = corr
            found_byte = k

    best_key.append(found_byte)

print(bytes(best_key).hex())
```

------------------------------------------------------------------------

## 4. Giai đoạn 2: Giải mã Flag

File: `decrypt_flag.py`

``` python
from Crypto.Cipher import AES
import binascii

KEY_HEX = "66dce15fb33deacb5c0362f30e95f52e"
key_bytes = binascii.unhexlify(KEY_HEX)

with open('encrypted_flag.bin', 'rb') as f:
    full_data = f.read()

iv = full_data[:16]
ciphertext = full_data[16:]

cipher = AES.new(key_bytes, AES.MODE_CBC, iv=iv)
decrypted = cipher.decrypt(ciphertext)

print(decrypted.decode('utf-8', errors='ignore').strip())
```

------------------------------------------------------------------------

## 5. Tổng kết

-   **Key:** `66dce15fb33deacb5c0362f30e95f52e`
-   **Flag:** `texsaw{d1ffer3nti&!_p0w3r_@n4!y51s}`

------------------------------------------------------------------------

## 🎯 Insight quan trọng

-   AES mạnh về toán học nhưng yếu khi triển khai phần cứng
-   Side-channel attack khai thác rò rỉ vật lý, không phá thuật toán
