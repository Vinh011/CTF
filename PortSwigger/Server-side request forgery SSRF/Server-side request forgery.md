# Server-side Request Forgery (SSRF)

Server-Side Request Forgery (SSRF) là lỗ hổng bảo mật web cho phép kẻ tấn công khiến server thực hiện các request đến địa chỉ tùy ý, thay vì chỉ các địa chỉ hợp lệ mà ứng dụng cho phép.

Thay vì browser gửi request đến server, ta lừa chính server gửi request đến nơi ta muốn — kể cả các hệ thống nội bộ mà bên ngoài không thể truy cập trực tiếp.

---

## Danh sách Lab

- [Lab: Basic SSRF against the local server](#lab-basic-ssrf-against-the-local-server)
- [Lab: Basic SSRF against another back-end system](#lab-basic-ssrf-against-another-back-end-system)
- [Lab: Blind SSRF with out-of-band detection](#lab-blind-ssrf-with-out-of-band-detection)
- [Lab: SSRF with blacklist-based input filter](#lab-ssrf-with-blacklist-based-input-filter)
- [Lab: SSRF with filter bypass via open redirection vulnerability](#lab-ssrf-with-filter-bypass-via-open-redirection-vulnerability)

---

## Lab: Basic SSRF against the local server

<img width="940" height="381" alt="image" src="https://github.com/user-attachments/assets/346ec36d-d098-47d5-881f-e9720aa6afc5" />


Bài này bị lỗi ở chỗ kiểm tra hàng tồn kho.

<img width="940" height="425" alt="image" src="https://github.com/user-attachments/assets/f6e0506a-7d82-4b41-9809-71869e2e6517" />


Ta bấm **View details** để check stock.

<img width="940" height="470" alt="image" src="https://github.com/user-attachments/assets/205bb37c-7a0f-45d0-a9c9-41ab470f1a9d" />


Ta thấy request của tính năng này có api trỏ đến chỗ kiểm tra số lượng.

<img width="940" height="689" alt="image" src="https://github.com/user-attachments/assets/39372b5d-97c4-4061-8999-05bc6dbdfa0b" />


Ta thay phần này bằng endpoint khác `http://localhost/admin` để truy cập.

<img width="940" height="335" alt="image" src="https://github.com/user-attachments/assets/15157cb4-41d1-4f07-9c58-9766086c3214" />


Ta đã thay đổi api và gửi request, thấy trang admin trả về có api xóa user.

<img width="940" height="331" alt="image" src="https://github.com/user-attachments/assets/91e11630-ebe4-45fe-930e-246ca4e9dad5" />


Ta thay api xóa user này vào để xóa và hoàn thành lab.

---

## Lab: Basic SSRF against another back-end system

<img width="940" height="381" alt="image" src="https://github.com/user-attachments/assets/c8982814-b937-4e52-8976-871cbcff43bd" />


Bài này bị lỗi SSRF ở chức năng kiểm tra tồn kho, ta khai thác với IP `192.168.0.x:8080`.

<img width="940" height="456" alt="image" src="https://github.com/user-attachments/assets/4e74915c-56fb-444c-918d-d60e618f566d" />


Vào **View details** để check stock.

<img width="940" height="467" alt="image" src="https://github.com/user-attachments/assets/33822124-39b7-4647-8b01-5e2cd6866591" />


Thực hiện tính năng này và bắt request trong Burp Suite.

<img width="940" height="719" alt="image" src="https://github.com/user-attachments/assets/8ae96fe5-c17b-442e-a796-e8fc8708a247" />


Ta thấy nó có api trỏ đến endpoint check số lượng, ta thay đổi thành dải IP `192.168.0.X:8080` như đề bài.

`X` là số IP ẩn ta cần tìm. Để tìm số này nhanh ta dùng tính năng **Intruder** của Burp Suite.

<img width="940" height="576" alt="image" src="https://github.com/user-attachments/assets/953ee0fb-70df-4f66-aedf-51693144683f" />


<img width="940" height="499" alt="image" src="https://github.com/user-attachments/assets/f257fe26-81a8-4d28-85da-1a2326223feb" />


Ta phát hiện IP cần tìm là `22` và đã truy cập được trang admin như trong phản hồi.

<img width="940" height="542" alt="image" src="https://github.com/user-attachments/assets/e900062c-7d75-45d1-948d-1115201c79df" />


Thay URL xóa user vào request `stockApi`, nhấn gửi và hoàn thành lab.

---

## Lab: Blind SSRF with out-of-band detection

<img width="940" height="531" alt="image" src="https://github.com/user-attachments/assets/09cdc6fc-6d3e-4f28-8378-3e00aa1a7803" />


Bài này bị lỗ SSRF, server tự gửi request ra ngoài.

<img width="940" height="417" alt="image" src="https://github.com/user-attachments/assets/0e3d8034-4fc7-40bd-9efb-f9a8ebd2a074" />


Ta bấm vào **View details** để kiểm tra từng request.

<img width="940" height="666" alt="image" src="https://github.com/user-attachments/assets/a17577e7-aca9-42da-af58-7a013af5191f" />


Ta kiểm tra request này và thấy header **Referer**. Referer là một HTTP header mà trình duyệt tự động gửi kèm theo mỗi request, để cho server biết người dùng đang đến từ trang nào.

Bài này có lỗi đó, ta sẽ sử dụng tính năng **Collaborator** của Burp Suite Pro để tạo 1 URL — URL này có thể nhận request tới nó.

<img width="940" height="432" alt="image" src="https://github.com/user-attachments/assets/3ab8e7e1-b437-47be-a28c-2d2625206de5" />


Ta thực hiện copy URL này rồi thay đổi giá trị của `Referer` bằng URL vừa tạo.

<img width="940" height="666" alt="image" src="https://github.com/user-attachments/assets/2925c8ce-08f2-4e51-8b13-2d054151900b" />


Sau khi thay đổi giá trị của `Referer` thì gửi rồi chuyển sang tab **Collaborator** để check.

<img width="940" height="312" alt="image" src="https://github.com/user-attachments/assets/cced5fc1-7487-449e-ac73-d369be025a30" />


Đã phát hiện server gửi request đến URL tôi tạo và đã hoàn thành lab.

---

## Lab: SSRF with blacklist-based input filter

<img width="940" height="431" alt="image" src="https://github.com/user-attachments/assets/2cd40ea1-767a-43dd-82de-56bf57c84fa7" />


Bài này có lỗi SSRF ở chỗ kiểm tra hàng tồn kho, nhưng nó đã được fix bằng cách thiết lập danh sách đen để chặn.

Cần vượt qua để truy cập vào URL: `http://localhost/admin`.

<img width="940" height="484" alt="image" src="https://github.com/user-attachments/assets/82bce358-9124-4f84-a137-e5cd43bbfa52" />


Truy cập và vào **View details**.

<img width="940" height="498" alt="image" src="https://github.com/user-attachments/assets/55d548ff-cdd6-499e-9e59-467e3f2fbd68" />


Sau khi vào View details ta thực hiện tiếp **Check stock** để xem request có lỗi ở chỗ này.

<img width="940" height="534" alt="image" src="https://github.com/user-attachments/assets/e586db08-6522-440d-ab7b-16f8b3a3a38e" />


Thử thay vào `stockApi` và toàn bị chặn:

- `localhost`
- `127.0.0.1`
- `admin`

<img width="940" height="353" alt="image" src="https://github.com/user-attachments/assets/b15bb555-7f02-4780-8718-7693a057b5dc" />


Ta có cách bypass là dùng IP viết tắt `127.1` của `127.0.0.1` — khi dùng IP viết tắt này thì server tự điền `0` vào giữa và được `127.0.0.1`.

<img width="940" height="363" alt="image" src="https://github.com/user-attachments/assets/02926aa4-115c-4c2f-a934-f9634bcd4aa9" />


Và đã bypass được. Tiếp theo ta truy cập trang admin nhưng nó lại bị chặn — có vẻ như server vừa chặn IP kia vừa chặn từ `admin`.

<img width="940" height="334" alt="image" src="https://github.com/user-attachments/assets/7495f8e6-b8f9-4462-af84-bc1fb40f0cf6" />


Ta sẽ encode 2 lần chữ `a` của từ admin:

```
a → %61 → %2561
```

Kết quả URL bypass: `http://127.1/%2561dmin`

<img width="940" height="511" alt="image" src="https://github.com/user-attachments/assets/cef258b1-7dc9-4ab2-9530-ef86ab52f171" />


Bypass thành công, giờ ta thay URL xóa user vào là hoàn thành lab.

---

## Lab: SSRF with filter bypass via open redirection vulnerability

<img width="940" height="406" alt="image" src="https://github.com/user-attachments/assets/ded96247-6c4a-4015-8aed-2142a50946fb" />

Bài này bị lỗi ở chỗ kiểm tra hàng tồn kho và phần chuyển hướng sản phẩm.

Tìm cách bypass whitelist để hoàn thành lab.

<img width="940" height="511" alt="image" src="https://github.com/user-attachments/assets/749e303b-d38d-4c24-9de4-7f33443113d7" />

Vào trang ta bấm **View details** để xem sản phẩm.

<img width="940" height="505" alt="image" src="https://github.com/user-attachments/assets/a8228776-e554-4259-a64c-4f1d7232c7b2" />

Vào xem thì ta thấy có 2 tính năng là kiểm tra số lượng và xem sản phẩm tiếp theo (**Next product**).

<img width="940" height="525" alt="image" src="https://github.com/user-attachments/assets/df5dbafd-51f8-4593-9f64-3c6b0ad7cd8c" />

Xem request của tính năng **Check stock** ta thấy `stockApi` đang gán đường dẫn của sản phẩm.

Ta thử thay thành `http://192.168.0.12:8080/admin` nhưng thất bại vì whitelist chặn.

<img width="940" height="364" alt="image" src="https://github.com/user-attachments/assets/93b2a199-aabe-4a77-97d8-602bd7062430" />

Ta kiểm tra request của **Next product** và thấy nó có thêm tham số `path` ở chỗ GET.

<img width="940" height="479" alt="image" src="https://github.com/user-attachments/assets/c351cd60-b481-4594-9070-25edee22d3a3" />

Thử thay giá trị của `path` bằng `http://192.168.0.12:8080/admin` để check và phát hiện nó có thể truy cập trang admin bằng cách này — server thực hiện open redirect theo giá trị `path`.

Ở phần request check stock trước ta thấy nó có endpoint `stockApi` để gọi URL. Ta thử dùng URL của endpoint **Next product** (kèm `path` trỏ đến admin) gán vào `stockApi`.

<img width="940" height="333" alt="image" src="https://github.com/user-attachments/assets/c4283d94-7598-4cc3-9a28-84a7a9546001" />

Thành công đến được trang admin, từ đây ta thay URL để xóa user và hoàn thành lab.

<img width="940" height="366" alt="image" src="https://github.com/user-attachments/assets/b0c24eb1-f8ec-400d-98d0-852d0492159c" />
