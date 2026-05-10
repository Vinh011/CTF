# Write up: Torrent Analyze

<img width="1412" height="779" alt="image" src="https://github.com/user-attachments/assets/03adaef7-591c-47f9-b7ba-3eb5e70bee20" />

Đề bài miêu tả một nhân viên tải gì đó, và bắt tìm tên file đã tài về !

---

Mở wireshark để xem giao thức được sử dụng và thấy nó sử dụng đa số là **BitTorrent DHT**

<img width="1207" height="279" alt="image" src="https://github.com/user-attachments/assets/d6938310-29ad-4d29-b85d-cf65e6fab33d" />

Giao thức này có thông tin **info_hash: e2467cbf021192c241367b892230dc1e05c0580e**. info_hash là kết quả của hàm SHA-1 áp dụng lên toàn bộ phần info trong file .torrent
<img width="1919" height="1019" alt="image" src="https://github.com/user-attachments/assets/adbd4267-6831-4690-9bb1-a52cfd411458" />

Tra cứu mã này trên ViusTotal và biết đc tên file như trong **hint** của đề bài.

<img width="1841" height="630" alt="image" src="https://github.com/user-attachments/assets/406a371a-7da3-4351-ad85-67fc5fe9a207" />

---

### Flag: **picoCTF{ubuntu-19.10-desktop-amd64.iso}**
