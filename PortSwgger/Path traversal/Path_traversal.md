# Path traversal

Kẻ tấn công thao túng đường dẫn file trong input để truy cập file ngoài thư mục cho phép. Thay vì chỉ đọc file hợp lệ như `/var/www/images/cat.jpg`, kẻ tấn công chèn ký tự `../` để leo lên thư mục cha và đọc file nhạy cảm như `/etc/passwd`.

**Nguyên nhân:** server ghép input người dùng thẳng vào đường dẫn file mà không kiểm tra hay làm sạch.

```python
filename = request.args.get("file")
path = "./uploads/" + filename
```

---

## File path traversal, simple case

<img width="940" height="463" alt="image" src="https://github.com/user-attachments/assets/7b67586c-3fd5-4b8e-a140-44bbad4dc943" />


Bài chứa lỗ hổng path traversal trong chức năng hiển thị ảnh sản phẩm. Server lấy tên file từ tham số `filename` và đọc trực tiếp từ thư mục `/var/www/images/`. Mục tiêu: đọc file `/etc/passwd`.

Bắt request load ảnh sản phẩm trong Burp.

<img width="940" height="446" alt="image" src="https://github.com/user-attachments/assets/861fda13-fd43-45c6-bd69-9504334333fb" />


Ném sang Repeater, đổi giá trị `filename` thành:

```
filename=../../../etc/passwd
```

<img width="940" height="382" alt="image" src="https://github.com/user-attachments/assets/7e8ecf47-0a75-41eb-b949-4d8ec2ae862c" />


Mỗi `../` leo lên một cấp thư mục — cần 3 lần để thoát khỏi `/var/www/images/` ra tới `/`, sau đó đọc `etc/passwd`. Response trả về nội dung file `/etc/passwd` chứa danh sách tài khoản hệ thống → lab Solved.

- `..` : thư mục cha (lên một cấp)
- `/` : phân cách thư mục
- `../../../` : leo lên 3 cấp từ `/var/www/images/` ra `/`

---

## Lab: File path traversal, traversal sequences blocked with absolute path bypass

<img width="940" height="588" alt="image" src="https://github.com/user-attachments/assets/4c2d47cc-0f96-4fe0-8b46-bcfd01acf532" />


Bài này server chặn các chuỗi `../` — nếu bạn gửi `../../../etc/passwd` sẽ bị block. Tuy nhiên server không kiểm tra đường dẫn tuyệt đối.

Thay vì dùng `../` để leo thư mục, dùng thẳng đường dẫn tuyệt đối:

```
filename=/etc/passwd
```

<img width="940" height="332" alt="image" src="https://github.com/user-attachments/assets/1e0900f3-faf4-4a6d-8160-770f7b442bad" />


Server nhận `/etc/passwd`, không thấy `../` nên không block, nhưng vẫn đọc thẳng file đó trả về. Response chứa nội dung `/etc/passwd` → lab Solved.

- Đường dẫn tương đối: `../../../etc/passwd` (leo từ vị trí hiện tại)
- Đường dẫn tuyệt đối: `/etc/passwd` (chỉ thẳng từ root `/`)

---

## Lab: File path traversal, traversal sequences stripped non-recursively

<img width="940" height="617" alt="image" src="https://github.com/user-attachments/assets/de6511ef-7a34-4c92-be21-82fd9f23d9d7" />


Bài này server lọc bỏ chuỗi `../` khỏi input — nhưng chỉ lọc một lần, không lọc lại sau khi đã xóa.

Lợi dụng điểm yếu: nhúng `../` vào giữa một chuỗi dài hơn để sau khi lọc vẫn còn lại `../`:

```
filename=....//....//....//etc/passwd
```

<img width="940" height="352" alt="image" src="https://github.com/user-attachments/assets/ad7dcf47-75eb-4788-a37c-ee8d9fc228ce" />


Server thấy `....//`, xóa `../` ở giữa, còn lại `../`. Kết quả sau lọc thành `../../../etc/passwd`, leo thư mục thành công, đọc được `/etc/passwd` → lab Solved.

---

## Lab: File path traversal, traversal sequences stripped with superfluous URL-decode

<img width="940" height="649" alt="image" src="https://github.com/user-attachments/assets/a0ca87e0-c184-4c72-84d0-7a1e13742936" />


Bài này server lọc `../` nhưng lọc **trước khi** URL-decode. Nếu encode `../` thành `..%2F` thì server lọc không phát hiện ra, sau đó server tự decode lại thành `../` trước khi đọc file.

Encode `../` theo URL encoding: `..%2F..%2F`

Payload:

```
filename=..%252f..%252f..%252fetc/passwd
```

Server lọc không thấy `../` nên cho qua, server decode thành `../`, leo thư mục đọc `/etc/passwd` → lab Solved.

<img width="940" height="352" alt="image" src="https://github.com/user-attachments/assets/bdf30e92-d8b9-44b9-bd27-4f7d6b6752b1" />


---

## Lab: File path traversal, validation of start of path

Bài này server kiểm tra input phải bắt đầu bằng `/var/www/images/` — nếu không có prefix này thì bị block.

Server chỉ kiểm tra phần đầu chuỗi chứ không kiểm tra phần còn lại, nên ta giữ nguyên prefix hợp lệ rồi leo thư mục từ đó:

Payload:

```
filename=/var/www/images/../../../etc/passwd
```

<img width="940" height="352" alt="image" src="https://github.com/user-attachments/assets/30fdd329-49a7-4aa6-83a3-a1602b5999f9" />


Server thấy bắt đầu bằng `/var/www/images/` hợp lệ, cho qua. Server xử lý `../../../` leo lên 3 cấp ra `/`, đọc `etc/passwd` → lab Solved.

---

## Lab: File path traversal, validation of file extension with null byte bypass

<img width="940" height="610" alt="image" src="https://github.com/user-attachments/assets/b4983a05-f1ea-4a29-ac0a-9fc7e3220a6f" />


Bài này server kiểm tra input phải kết thúc bằng `.jpg` — nếu không có extension ảnh thì bị block.

Lợi dụng null byte (`%00`) — đây là ký tự kết thúc chuỗi trong C/C++. Khi server kiểm tra extension, nó thấy `.jpg` ở cuối nên pass. Nhưng khi hệ điều hành đọc file, nó gặp null byte và dừng tại đó, bỏ qua phần `.jpg` phía sau:

Payload:

```
filename=../../../etc/passwd%00.jpg
```

<img width="940" height="350" alt="image" src="https://github.com/user-attachments/assets/387e355b-7db0-43ce-ba4f-cb01b717e5ca" />
