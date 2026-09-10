# Authentication

## Lỗ hổng này là gì?

Authentication (xác thực) là cơ chế ứng dụng dùng để xác minh danh tính người dùng. Với mô hình đăng nhập bằng username/password, hệ thống mặc định rằng chỉ chủ tài khoản mới biết password của mình, nên chỉ cần nhập đúng cặp username/password là được công nhận là đúng người. Vì vậy độ an toàn của cả hệ thống phụ thuộc hoàn toàn vào hai điều: password phải thực sự bí mật, và cơ chế kiểm tra đăng nhập không được vô tình để lộ thêm thông tin nào khác. Chỉ cần một trong hai điều đó bị phá vỡ là phát sinh lỗ hổng.

Hai dạng cụ thể trong nhóm lỗ hổng này:

- **Brute-force attack**: kẻ tấn công thử hàng loạt cặp username/password (thường dùng wordlist và công cụ tự động như Burp Intruder) cho tới khi tìm được thông tin đăng nhập hợp lệ. Việc tự động hoá giúp thực hiện được số lượng lớn lượt thử trong thời gian ngắn.
- **Username enumeration**: kẻ tấn công không đoán password ngay mà đoán xem một username có tồn tại hay không, dựa vào sự khác biệt (dù rất nhỏ) trong phản hồi của server. Một khi đã có danh sách username hợp lệ, việc brute-force password sẽ nhanh hơn rất nhiều vì không cần thử với những username không tồn tại.

## Tại sao lỗ hổng này tồn tại?

Nguyên nhân gốc rễ thường đến từ việc server phản hồi **không nhất quán** giữa trường hợp "sai" và "đúng một phần":

- Message lỗi, mã trạng thái HTTP, hoặc độ dài (length) response bị khác nhau tinh vi giữa "username sai" và "username đúng nhưng password sai" — dù giao diện hiển thị cho người dùng trông giống hệt nhau.
- Thời gian xử lý (response timing) khác nhau: nếu username sai, server có thể bỏ qua luôn bước hash/so sánh password nên trả lời nhanh hơn so với khi username đúng.
- Cơ chế chống brute-force (khóa tài khoản, khóa IP sau N lần sai) bị cài đặt sai logic — ví dụ bộ đếm bị reset không đúng cách, hoặc chỉ khóa theo địa chỉ IP nên có thể bị bypass bằng cách giả mạo IP qua header như `X-Forwarded-For`.

Vì các sai sót này thường rất nhỏ (một vài mili-giây, một vài byte trong response), pentester cần dùng công cụ như Burp Suite Intruder để tự động gửi hàng loạt request và so sánh phản hồi mới phát hiện ra được.

---

## Lab: Username enumeration via subtly different responses

Bài này yêu cầu brute-force username và password, thay vì xem trường length khác biệt thì bài muốn mình xem response khác biệt khi login. Để làm bài này mình thêm trường lấy nội dung từ response để xem sự khác biệt, như dưới ảnh thì tìm được username.

<img width="1125" height="500" alt="image" src="https://github.com/user-attachments/assets/2334922f-337e-42a0-aeeb-37913e550c6e" />

<img width="940" height="501" alt="image" src="https://github.com/user-attachments/assets/b0f1bed6-5490-48d1-9ec9-0f239ba9003a" />


Sau khi tìm được username thì brute-force password và tìm trường length khác cái còn lại.

<img width="940" height="497" alt="image" src="https://github.com/user-attachments/assets/a0a0a440-1eca-47f7-8193-7ad2129b063e" />

---

## Lab: Username enumeration via response timing

Bài này đã fix thêm là chống brute-force từ 1 IP, bypass bằng cách sử dụng header `X-Forwarded-For` để có thể thay đổi IP cho mỗi request, và bài có nhắc tới response timing và cho tài khoản và mật khẩu để test thời gian response từ server (username sai thì không kiểm tra mật khẩu nên thời gian ngắn), từ đó có thể xem trường response time để xem sự khác biệt.

<img width="940" height="496" alt="image" src="https://github.com/user-attachments/assets/f68a5210-374b-4a8a-ac26-75d0140d8461" />

Sau khi tìm được username thì brute-force password và xem sự khác biệt từ trường length.

<img width="940" height="497" alt="image" src="https://github.com/user-attachments/assets/d8509fb6-be8b-4648-8114-10ea09c977b3" />

---

## Lab: Broken brute-force protection, IP block

Bài này fix brute-force password bằng cách cứ login sai 3 lần là khóa 1 phút, và đề bài có username và password để test. Cứ khi login sai 2 lần mà lần thứ 3 login đúng thì được reset limit. Biết thế nên tôi tạo list pass cứ 2 lần login sai thì tôi chèn vào tài khoản và mật khẩu đúng vào, rồi quan sát trường length của username cần tìm.

<img width="940" height="501" alt="image" src="https://github.com/user-attachments/assets/243f6437-5c23-407b-ab99-f310898747e0" />
