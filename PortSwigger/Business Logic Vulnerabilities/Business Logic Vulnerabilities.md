# Business Logic Vulnerabilities

Business logic vulnerability là lỗ hổng xảy ra khi luồng xử lý nghiệp vụ (business flow) của ứng dụng bị lạm dụng theo cách mà nhà phát triển không lường trước, dù code "chạy đúng" về mặt kỹ thuật (không có SQLi, XSS, buffer overflow...). Nói cách khác: hệ thống không bị "hack" bằng kỹ thuật injection, mà bị khai thác bằng cách dùng chức năng hợp lệ theo trình tự/giá trị không hợp lệ.

**Map toàn bộ luồng nghiệp vụ:**

Duyệt toàn bộ ứng dụng như người dùng bình thường, ghi lại từng bước: đăng ký → đăng nhập → tìm sản phẩm → thêm giỏ → thanh toán → xác nhận.

---

## Danh sách Lab

- [Lab: Excessive trust in client-side controls](#lab-excessive-trust-in-client-side-controls)
- [Lab: High-level logic vulnerability](#lab-high-level-logic-vulnerability)
- [Lab: Inconsistent security controls](#lab-inconsistent-security-controls)
- [Lab: Flawed enforcement of business rules](#lab-flawed-enforcement-of-business-rules)
- [Lab: Low-level logic flaw](#lab-low-level-logic-flaw)
- [Lab: Inconsistent handling of exceptional input](#lab-inconsistent-handling-of-exceptional-input)
- [Lab: Weak isolation on dual-use endpoint](#lab-weak-isolation-on-dual-use-endpoint)
- [Lab: Insufficient workflow validation](#lab-insufficient-workflow-validation)
- [Lab: Authentication bypass via flawed state machine](#lab-authentication-bypass-via-flawed-state-machine)
- [Lab: Infinite money logic flaw](#lab-infinite-money-logic-flaw)

---

## Lab: Excessive trust in client-side controls

<img width="940" height="564" alt="image" src="https://github.com/user-attachments/assets/64c7e725-e421-4f7c-bc8c-23cd7823931c" />


Bài này cho biết server không kiểm tra thông tin mà client gửi lên. Mua áo Lightweight l33t leather jacket để giải bài này.

Đăng nhập tài khoản có sẵn: `wiener:peter`.

<img width="940" height="795" alt="image" src="https://github.com/user-attachments/assets/d2432620-758b-478f-ac65-29834338637f" />


Sau khi đăng nhập ta có 100$ để mua hàng.

<img width="940" height="661" alt="image" src="https://github.com/user-attachments/assets/3c47a0f6-1edc-459e-8b3f-cfdf2ecd4fae" />


Nhưng nhìn giá bán áo Lightweight l33t leather jacket lại tận 1337$ vượt quá số tiền hiện có.

<img width="940" height="532" alt="image" src="https://github.com/user-attachments/assets/74b25b15-e0dc-4c35-9c3d-c64a2c340459" />


Đề bài có nhắc tới xác thực có vẻ liên quan đến logic nào đó khi mua hàng, tôi thực hiện mua hàng và bắt request để kiểm tra.

<img width="940" height="376" alt="image" src="https://github.com/user-attachments/assets/f098311b-3116-44cd-92e7-e9e3e01692a5" />


Tôi thực hiện thêm vào giỏ hàng rồi bắt request và thấy các thông tin như:

- `productId`: id sản phẩm.
- `quantity`: số lượng sản phẩm.
- `price`: giá sản phẩm.

Ta thấy khi gửi lên sẽ có trường giá tiền là `133700`, tôi thử đổi sang giá tiền khác trước khi gửi lên server xem có phản hồi gì.

<img width="940" height="409" alt="image" src="https://github.com/user-attachments/assets/1cbdb90d-4fb5-4972-8559-037bd49a045c" />


Sau khi chỉnh sửa thành giá tiền khác và gửi lên thì thấy server chấp nhận.

<img width="940" height="408" alt="image" src="https://github.com/user-attachments/assets/fe5793f6-2419-4194-84b7-aef893fc36c7" />


Kiểm tra lại giỏ hàng khi sửa giá tiền thì thấy giá tiền như đã sửa, từ đây tôi có thể mua hàng.

<img width="940" height="578" alt="image" src="https://github.com/user-attachments/assets/1fc72908-d698-44b5-b39b-a2e16be5856e" />


---

## Lab: High-level logic vulnerability

<img width="940" height="571" alt="image" src="https://github.com/user-attachments/assets/5cdcac17-9667-4c5d-8167-0d153107ef91" />


Tôi login vào tài khoản đã cho ở đề bài: `wiener:peter`. Sau khi login được tặng 100$.

<img width="940" height="450" alt="image" src="https://github.com/user-attachments/assets/5580e746-757e-444f-8a6c-c55141f27cd3" />


Thực hiện chức năng thêm vào giỏ hàng để bắt request thấy không còn để giá tiền trong request nữa.

<img width="861" height="653" alt="image" src="https://github.com/user-attachments/assets/9bad4c7e-86ba-4c21-ad43-873e19c1359f" />


Thực hiện chức năng thanh toán và được phản hồi tiền không đủ để thanh toán.

<img width="940" height="482" alt="image" src="https://github.com/user-attachments/assets/c0e0d999-76e0-4bd0-8932-7df6f8d5d348" />


Quay lại request thêm vào giỏ hàng ta thấy có: id sản phẩm, số lượng... Thử sửa đổi giá trị ở chỗ này xem.

<img width="881" height="746" alt="image" src="https://github.com/user-attachments/assets/bcf1adfc-dc0d-4a4e-9f1d-fb4f5240d632" />


Tôi đổi số lượng thành 5 và thấy số lượng trong giỏ hàng tăng lên và giá tiền cũng nhân 5.

<img width="940" height="451" alt="image" src="https://github.com/user-attachments/assets/6428c370-d8f1-44f2-8b39-d9d3ce5b0201" />


Có vẻ server chấp nhận chỉnh sửa này từ phía client, vậy thì tôi thử giảm nó về số âm xem số lượng và giá tiền có giảm theo hay không.

<img width="888" height="728" alt="image" src="https://github.com/user-attachments/assets/f8f3c7a4-1e56-4c1b-a9a9-1426b4b8e87a" />


Tôi đã giảm số lượng về giá trị âm (`-1`) và thấy trong giỏ hàng được hiển thị là -1 sản phẩm với số tiền âm. Tôi đã thử thanh toán và được báo là giá sản phẩm phải lớn hơn 0.

<img width="940" height="612" alt="image" src="https://github.com/user-attachments/assets/ca4e3b6d-11a3-4460-828c-c6302204b2e6" />


Vậy server vừa chấp nhận số tiền âm và dương thì khi yêu cầu giá tiền hơn 0 mới có thể thanh toán thì tôi thêm vài sản phẩm với số lượng âm sao cho giá tiền tổng lớn hơn 0 và nhỏ hơn số tiền hiện có. Sau khi thêm vài sản phẩm thì tôi đã thanh toán thành công.

<img width="940" height="342" alt="image" src="https://github.com/user-attachments/assets/a50d0991-7917-45de-a281-9a172933a783" />


---

## Lab: Inconsistent security controls

<img width="940" height="516" alt="image" src="https://github.com/user-attachments/assets/743c8f45-0322-4279-9c17-355d5ec0b9a1" />


Đề bài cho biết lỗ hổng ở trang quản trị.

Vào trang thì có chức năng Email client, click vào thì thấy phải dùng email được cung cấp để đăng kí mới xác thực được.

<img width="940" height="325" alt="image" src="https://github.com/user-attachments/assets/82e5f61e-3344-4136-82e3-cf2296058077" />


Tôi lấy domain email này để đăng kí tài khoản mới.

<img width="940" height="567" alt="image" src="https://github.com/user-attachments/assets/c932b66e-33fb-4358-82f3-377b9933f5ba" />


Sau khi thực hiện đăng kí tài khoản, thấy có phản hồi là cần vào link để xác nhận.

<img width="940" height="394" alt="image" src="https://github.com/user-attachments/assets/2cd07a1f-9f07-47bf-86eb-4a7a1ef867d9" />


Vào Email client sau khi đăng kí thì thấy link xác nhận, nên tôi bấm vào để xác nhận đăng kí tài khoản.

<img width="940" height="344" alt="image" src="https://github.com/user-attachments/assets/54580d5a-b16e-4d1b-86a6-6e4b1eb1db04" />


Ta được phản hồi đăng kí thành công.

<img width="940" height="304" alt="image" src="https://github.com/user-attachments/assets/eb8de2b4-64ad-4087-8fb2-9e5e73196784" />


Quay lại trang đăng nhập và thực hiện đăng nhập vào tài khoản vừa đăng kí thành công và thấy ô **update email**.

<img width="940" height="441" alt="image" src="https://github.com/user-attachments/assets/1f70ed71-2c1a-41eb-a0ba-def6cd2f9b2f" />


Quay lại trang đăng kí ta thấy admin có tên là `DontWannaCry` thì đăng kí tài khoản bằng domain email `@dontwannacry.com`.

<img width="940" height="554" alt="image" src="https://github.com/user-attachments/assets/206fc723-cdca-419b-acc0-9bd20cf0360c" />


Nếu đăng kí tài khoản bằng domain email này thì không thể xác thực để đăng kí, nhưng sau khi đăng nhập thì ta có chức năng update email, vậy tôi thử update thành domain email của DontWannaCry.

<img width="940" height="318" alt="image" src="https://github.com/user-attachments/assets/a3c19a92-63bf-4657-9212-a4a824d9eb0e" />


Sau khi update email thì server không xác thực và chấp nhận luôn, và tôi đã có tính năng **Admin panel**, tôi thực hiện xóa tài khoản carlos như yêu cầu và thành công.

<img width="940" height="368" alt="image" src="https://github.com/user-attachments/assets/88a475a7-c628-479b-9aa7-8f54c4c6df13" />


---

## Lab: Flawed enforcement of business rules

<img width="940" height="547" alt="image" src="https://github.com/user-attachments/assets/de0fa2b6-2c8f-44aa-b8b9-26cfc7f2b184" />


Đề bài cho biết cần mua áo để giải bài.

<img width="940" height="373" alt="image" src="https://github.com/user-attachments/assets/d4db20d3-899d-44d2-bafd-b19b3753a3db" />


Mới vào đã thấy mã giảm giá: `NEWCUST5` — tôi thêm sản phẩm vào giỏ hàng để áp mã giảm giá xem.

<img width="940" height="679" alt="image" src="https://github.com/user-attachments/assets/7f7b28f1-2fc4-4315-8948-72a573326ee0" />

Sau khi thêm vào giỏ hàng, tôi đã áp mã giảm giá nhưng vẫn chưa đủ tiền để mua nên tôi áp thêm một lần nữa nhưng nó báo mã đã được áp dụng.

<img width="878" height="720" alt="image" src="https://github.com/user-attachments/assets/b4e7e481-b92e-4027-8ddf-01f1c9ca637b" />


Tôi đã thử đổi thành số âm nhưng lần này server không chấp nhận.

Lục lọi một hồi tôi phát hiện ô input, tôi nhập email của tài khoản mình vào.

<img width="940" height="328" alt="image" src="https://github.com/user-attachments/assets/37f84804-7caf-4489-a857-0493d61651db" />


Phát hiện có một mã giảm giá khác: `SIGNUP30`.

<img width="940" height="274" alt="image" src="https://github.com/user-attachments/assets/93c65f1e-cd8f-45c2-b804-88f53d2b8fe6" />


Tôi áp thêm vào và nó vẫn chưa giảm đủ để mua.

<img width="940" height="601" alt="image" src="https://github.com/user-attachments/assets/1ef03cf8-e25d-4adb-b03b-af9cbe1d378d" />


Tôi thử áp lại mã `SIGNUP30` lại một lần nữa và nó vẫn báo mã đã được áp dụng và không được giảm.

<img width="940" height="726" alt="image" src="https://github.com/user-attachments/assets/63f98c3f-1c04-4d15-9542-8ffeb3df7387" />

Vậy đã áp hai mã giảm giá nhưng vẫn chưa giảm đủ tiền để mua.

Tên đề bài có nhắc đến quy tắc gì đó, tôi thử áp lại mã `NEWCUST5` xem sao.

<img width="940" height="488" alt="image" src="https://github.com/user-attachments/assets/2a03ef8c-31b8-4fe4-9ab8-949480a1ae81" />


Sau khi áp lại mã `NEWCUST5` thì phát hiện nó nhận lại mã đã dùng sau khi dùng một mã khác, vậy tôi sẽ áp mã xen kẽ để giảm đến lúc có thể mua.

<img width="940" height="619" alt="image" src="https://github.com/user-attachments/assets/65a8e385-4487-4a99-adef-db75f0ffaba7" />


Sau khi áp dụng mã giảm đủ thì tôi đã mua thành công.

---

## Lab: Low-level logic flaw

<img width="940" height="447" alt="image" src="https://github.com/user-attachments/assets/cd5f9caf-30e7-4833-a137-ddc52585be66" />


Bài này có lỗ hổng ở input và cần mua áo để hoàn thành bài này. Bài này cho tài khoản sẵn để login: `wiener:peter`.

Login vào bài tôi thử thêm vào giỏ hàng rồi mua nhưng không đủ tiền, tôi đã thử cách như thêm số lượng âm nhưng nó không nhận.

<img width="878" height="783" alt="image" src="https://github.com/user-attachments/assets/5fc363b3-a9d8-4fd3-a79d-bacb0c5a48ad" />


Chỗ này tôi đã thử số lượng âm nhưng không thêm được vào giỏ hàng, vậy thử thêm số lượng cực lớn xem nó có bị tràn số nguyên hay không.

Tôi chuyển request này sang tab **Intruder** và sửa số lượng sản phẩm lớn, tôi đã thử để 999 nhưng được validate là số lượng không thể lớn hơn 99 nên tôi đã để số lượng 99 để tự động gửi request liên tục thêm vào giỏ hàng nhiều lần.

<img width="940" height="574" alt="image" src="https://github.com/user-attachments/assets/b98508a1-972a-4179-bbdc-00135dadd098" />


Chọn chế độ tấn công **Sniper attack** rồi **Start attack**.

<img width="940" height="499" alt="image" src="https://github.com/user-attachments/assets/0e3ef483-c811-4aaa-8b29-7cbb42ce0e75" />


Đang gửi request này liên tục cho đến khi đạt số lượng đủ lớn — giá tiền tổng của sản phẩm áo này bị tràn số nguyên và chuyển giá trị về âm.

<img width="940" height="619" alt="image" src="https://github.com/user-attachments/assets/19ac2838-d408-495c-9815-032667edf201" />


Ở đây shop không cho phép mua sản phẩm với giá tiền âm thì tôi thêm vài sản phẩm vào cho nó có số tiền dương nhưng vẫn đủ tiền để mua.

<img width="940" height="819" alt="image" src="https://github.com/user-attachments/assets/9fae5836-b2f1-48b4-92f9-b360394afc40" />


Tôi đã thực hiện mua nhiều ở một sản phẩm để cho nó bị tràn số ở biến lưu giá tiền sản phẩm này, sau đó tôi thêm số lượng áo cần mua cho vừa đủ số tiền đang có để thanh toán.

---

## Lab: Inconsistent handling of exceptional input

Hệ thống kiểm tra email có phải thuộc domain công ty không bằng cách tách chuỗi email ra để lấy phần domain. Nhưng khi email quá dài, đoạn code tách chuỗi phía backend xử lý khác với cách con người đọc khiến ta có thể giấu domain công ty vào giữa một chuỗi dài, và hệ thống vẫn nhận diện nhầm là domain hợp lệ.

<img width="940" height="450" alt="image" src="https://github.com/user-attachments/assets/35680627-995a-40a4-9026-d929542faca6" />


Vào trang và đăng kí tài khoản với một chuỗi email rất dài ta thấy email sau khi đăng kí bị cắt mất phần sau, đếm thì thấy có 255 ký tự — có thể server chỉ lưu 255 ký tự email.

<img width="940" height="425" alt="image" src="https://github.com/user-attachments/assets/a72dfa35-7df6-4485-9c6b-9a77a0545ed2" />


Vậy để có thể vào trang admin để xóa user thì cần đăng kí được bằng domain email `@dontwannacry.com` nhưng để kích hoạt được tài khoản thì ta phải dùng cả domain `@exploit-0add00f70312401a806cfc3501fe000e.exploit-server.net` để có thể kích hoạt.

<img width="940" height="373" alt="image" src="https://github.com/user-attachments/assets/ece84e2d-6bc9-47ba-8872-ccdd9bbf14ce" />


Như đã biết chuỗi email giới hạn là 255 ký tự nên tôi đã đăng kí email vừa đủ để có thể vừa xác thực được link vừa có domain `@dontwannacry.com` để có thể mở được trang admin và thực hiện xóa user.

<img width="940" height="381" alt="image" src="https://github.com/user-attachments/assets/bb72289a-2e77-4cfc-9085-f071606925e2" />

Sau khi đăng kí email thành công ta đã có trang **Admin panel**.

---

## Lab: Weak isolation on dual-use endpoint

<img width="940" height="418" alt="image" src="https://github.com/user-attachments/assets/2ae211fe-9dfa-495e-aa7c-04277e47c2cf" />


Bài này gây lỗ hổng ở chỗ endpoint thay đổi mật khẩu.

<img width="940" height="430" alt="image" src="https://github.com/user-attachments/assets/3d9d43f7-dd8e-49ad-9ab6-50b05092a261" />


Login vào tài khoản `wiener:peter` như đã cho ở đề bài để test các tính năng.

<img width="940" height="476" alt="image" src="https://github.com/user-attachments/assets/32d91859-b4f5-4d78-9273-cf550e29eafd" />


Sau khi login ta thấy có 2 tính năng là cập nhật email và thay đổi mật khẩu, ta dùng Burp Suite để bắt các request của các tính năng này.

<img width="940" height="394" alt="image" src="https://github.com/user-attachments/assets/e313968e-bc00-491a-b43a-cf2b20a60445" />


<img width="558" height="733" alt="image" src="https://github.com/user-attachments/assets/9ddd5340-db3c-4e02-83b7-c398e8106f65" />

Sau khi sử dụng Burp Suite để chặn HTTP request của chức năng đổi mật khẩu, ta quan sát được các tham số gồm `csrf`, `username`, `current-password`, `new-password-1` và `new-password-2`.

Đề bài nhắc đến cần tài khoản `administrator` để xóa người dùng `carlos` để hoàn thành lab, vậy ta tìm cách để có thể login vào tài khoản administrator.

<img width="940" height="683" alt="image" src="https://github.com/user-attachments/assets/48f219d7-4e42-46be-af28-719d919f1aaf" />


Thay vì đổi mật khẩu người dùng, tôi đổi mật khẩu administrator bằng cách thay giá trị của `username` bằng `administrator` và gửi request nhưng thấy server báo *"Current password is incorrect"*. Vậy tôi thử xóa trường `current-password` đi để kiểm tra xem server có kiểm tra kĩ phần này hay không.

<img width="940" height="683" alt="image" src="https://github.com/user-attachments/assets/4e9a243b-0442-4c42-95e8-a890a6507d87" />


Sau khi xóa trường mật khẩu cũ của tài khoản và thực hiện gửi thì phát hiện server chấp nhận thiếu trường này và gây ra lỗ hổng là có thể đổi mật khẩu người khác mà không cần xác thực. Từ đây ta đã đổi mật khẩu của administrator thành `a`.

<img width="940" height="472" alt="image" src="https://github.com/user-attachments/assets/c902e274-c2b5-4ad2-a477-ffd5884d64c9" />


Thực hiện login bằng tài khoản vừa đổi và đã có tính năng **Admin panel**, vào và xóa người dùng `carlos`.

---

## Lab: Insufficient workflow validation

<img width="940" height="516" alt="image" src="https://github.com/user-attachments/assets/f23bf8de-ed41-475a-a076-ceb0882eebf8" />


Bài này lỗi trong quy trình mua hàng.

<img width="940" height="424" alt="image" src="https://github.com/user-attachments/assets/f21394f3-6417-4d7d-a6e0-803a3993fdb7" />


Login vào tài khoản cho sẵn `wiener:peter` để test.

<img width="940" height="484" alt="image" src="https://github.com/user-attachments/assets/14e383a4-ea71-490a-92fc-62d6e864cc7a" />


Sau khi login ta thấy chiếc áo **Lightweight "l33t" Leather Jacket** cần mua để hoàn thành lab, nhưng nó có giá 1337$ và tài khoản chỉ có 100$.

<img width="940" height="453" alt="image" src="https://github.com/user-attachments/assets/3dceecf5-97e1-44a0-a4cd-77cfdc486a04" />


Ta thực hiện mua sản phẩm với giá ít hơn để test luồng hoạt động thanh toán.

<img width="746" height="328" alt="image" src="https://github.com/user-attachments/assets/b9b629aa-6b9a-4581-bbaf-9186ba769c8d" />


Sau khi thêm vào giỏ thì thực hiện thanh toán thành công, còn mua áo kia thì không đủ tiền.

Đề bài có nhắc tới luồng thanh toán vì vậy tôi kiểm tra lại các request trong Burp Suite để xem luồng hoạt động tính năng này.

<img width="940" height="376" alt="image" src="https://github.com/user-attachments/assets/78b6784d-fd99-409e-8053-cb8e030109d5" />


Sau khi thêm vào giỏ hàng và thực hiện thanh toán ta thấy nó có request check sản phẩm rồi gọi endpoint `/order`.

<img width="940" height="616" alt="image" src="https://github.com/user-attachments/assets/b9e5fa76-78c5-4136-bf16-aeb2cafacf8e" />


Kiểm tra kĩ hơn thì không thấy nó có check gì về sản phẩm trong giỏ hàng, nên tôi đã thêm áo kia vào để gọi luôn endpoint này.

<img width="940" height="364" alt="image" src="https://github.com/user-attachments/assets/d3531d83-e7e2-479d-abb9-9dfb77340fc1" />


---

## Lab: Authentication bypass via flawed state machine

<img width="940" height="412" alt="image" src="https://github.com/user-attachments/assets/eaa09f97-45f4-4d9d-98b3-590871476cc8" />


Bài này có lỗi thiếu sót trong quy trình đăng nhập.

<img width="940" height="409" alt="image" src="https://github.com/user-attachments/assets/47b77f1f-ea92-4605-a86b-3f00b7692e0e" />


Login với tài khoản được cho sẵn `wiener:peter`.

<img width="940" height="366" alt="image" src="https://github.com/user-attachments/assets/9f44e36a-60cc-4fd1-9237-f52ed23e7932" />


Sau khi login ta thấy phần chọn **role** cho tài khoản, thay vì chọn role thì có thể bỏ qua bước này bằng cách sử dụng tính năng **Intercept** trong Burp Suite để chuyển thẳng đến trang chính.

<img width="940" height="529" alt="image" src="https://github.com/user-attachments/assets/b890d40d-38da-4dd7-a7f7-7117204e0ffe" />


Ta dùng tính năng **Intercept** để bắt request GET để lấy phần xác thực role, thay vì **Forward** thì **Drop** nó để bỏ gói request này.

<img width="940" height="447" alt="image" src="https://github.com/user-attachments/assets/8f654e87-9750-4036-b6af-bcb95e722381" />


Sau khi drop request xác thực role đó, tôi quay lại trang chính và đã có trang **Admin panel**, vào xóa user `carlos` và hoàn thành lab.

<img width="940" height="429" alt="image" src="https://github.com/user-attachments/assets/2e3e2f14-2daf-4198-ad27-8c30c29a7ad1" />


---

## Lab: Infinite money logic flaw

<img width="940" height="465" alt="image" src="https://github.com/user-attachments/assets/972409b3-8304-4a2f-ac33-d882853ed0b4" />


Đề bài nói về lỗ hổng tiền vô hạn trong logic mua hàng.

<img width="940" height="462" alt="image" src="https://github.com/user-attachments/assets/98bb9dbb-cdbd-4d92-bc64-58368e8b5a59" />


Shop này khi mua 1 sản phẩm nó sẽ cho một mã code, mã code này áp vào thì sẽ được hoàn lại tiền.

<img width="940" height="408" alt="image" src="https://github.com/user-attachments/assets/6d94d8fd-862e-450a-a9ab-b65cfe381f60" />


Và shop còn cho ta mã giảm giá nữa.

<img width="940" height="355" alt="image" src="https://github.com/user-attachments/assets/732d814c-1bd4-4763-913b-490a6f79b7e6" />


Lợi dụng lỗ hổng này, ta vừa áp mã giảm giá vừa áp mã code để hoàn tiền — vì nó vẫn hoàn tiền theo giá gốc của sản phẩm — lặp đến khi đủ tiền mua áo.
