# Giai Đoạn 2: Linux Cho DevOps

> 8 bài học | Mỗi bài ~60 phút | Mục tiêu: Thao tác như một DevOps engineer trên server

---

# Bài 7: User, Group & Sudo

> ⏱️ Thời gian: ~60 phút (15p lý thuyết + 30p thực hành + 15p task)

## Mục tiêu
- Hiểu user/group trong Linux
- Dùng `sudo` để chạy lệnh admin
- Tạo user mới, đổi mật khẩu
- Biết DevOps thường dùng user riêng để deploy, không dùng root

## Lý thuyết

Linux là hệ điều hành **đa user**. Mỗi người có tài khoản riêng.

| Khái niệm | Ý nghĩa |
|-----------|---------|
| **root** | Superuser, quyền cao nhất |
| **user** | Người dùng thường (ví dụ: hoang) |
| **group** | Nhóm user, gán quyền theo nhóm |
| **sudo** | Cho phép user thường chạy lệnh admin |

> 🎯 DevOps thực tế: Không bao giờ chạy service bằng root. Luôn tạo user riêng (ví dụ: `deploy`, `appuser`).

## Thực hành từng bước

### Bước 1: Xem user hiện tại
```bash
whoami
id
```

Output:
```
hoang
uid=1000(hoang) gid=1000(hoang) groups=1000(hoang),4(adm),24(cdrom)...
```

Giải thích: `uid` = user ID, `gid` = group ID.

### Bước 2: Chuyển sang root
```bash
sudo su -
```

Nhập mật khẩu của bạn (khi gõ không hiện ký tự, đó là bình thường).

Prompt đổi thành `root@...#`.

```bash
whoami
exit
```

`exit` để thoát về user cũ.

### Bước 3: Chạy lệnh admin bằng sudo
```bash
sudo apt update
```

Chỉ cần thêm `sudo` trước lệnh cần quyền cao.

### Bước 4: Xem danh sách user trên hệ thống
```bash
cat /etc/passwd | head -10
```

Mỗi dòng là 1 user. Format:
```
username:password:UID:GID:fullname:home:shell
```

### Bước 5: Tạo user mới
```bash
sudo useradd -m -s /bin/bash devops_user
```

- `-m` = tạo thư mục home
- `-s /bin/bash` = shell mặc định

Kiểm tra:
```bash
id devops_user
```

Output:
```
uid=1001(devops_user) gid=1001(devops_user) groups=1001(devops_user)
```

### Bước 6: Đặt mật khẩu cho user mới
```bash
sudo passwd devops_user
```

Nhập mật khẩu mới 2 lần.

### Bước 7: Chuyển sang user mới
```bash
su - devops_user
whoami
pwd
```

Output:
```
devops_user
/home/devops_user
```

Thử tạo file:
```bash
touch test.txt
ls -la
exit
```

### Bước 8: Thêm user vào group sudo
```bash
sudo usermod -aG sudo devops_user
```

- `-aG` = append vào Group
- Giờ `devops_user` có thể dùng `sudo`

Kiểm tra:
```bash
id devops_user
```

Output có thêm `27(sudo)`.

### Bước 9: Xóa user (dọn dẹp)
```bash
sudo userdel -r devops_user
```

> ⚠️ `-r` = xóa cả home directory.

## Task cuối bài (15 phút)

1. Tạo user `deploy` với home directory.
2. Đặt mật khẩu `deploy123` cho user đó.
3. Đăng nhập vào user `deploy`, tạo file `deploy_note.txt` chứa nội dung "Deploy user ready".
4. Thoát về `hoang`, xóa user `deploy`.

## Đáp án / Gợi ý

```bash
# Task 1
sudo useradd -m -s /bin/bash deploy

# Task 2
sudo passwd deploy
# Nhập: deploy123

# Task 3
su - deploy
echo "Deploy user ready" > deploy_note.txt
cat deploy_note.txt
exit

# Task 4
sudo userdel -r deploy
```

## Lưu ý quan trọng
- `root` có quyền tuyệt đối. Một lệnh sai có thể phá hủy hệ thống. DevOps luôn dùng `sudo` thay vì đăng nhập root.
- File `/etc/sudoers` quản lý ai được dùng `sudo`. Chỉ sửa bằng `visudo`.
- Trên server production, mỗi người có user riêng, audit log ghi lại ai dùng `sudo`.

---

# Bài 8: Systemd & Service Management

> ⏱️ Thời gian: ~60 phút (15p lý thuyết + 30p thực hành + 15p task)

## Mục tiêu
- Hiểu `systemd` là gì
- Dùng `systemctl` để quản lý service
- Dùng `journalctl` để xem log service
- DevOps quản lý nginx, docker, ssh... đều bằng systemctl

## Lý thuyết

**Systemd** = hệ thống quản lý service của Linux hiện đại.

Một **service** = chương trình chạy nền (ví dụ: web server, database, SSH server).

| Lệnh | Ý nghĩa |
|------|---------|
| `systemctl status <service>` | Xem trạng thái |
| `systemctl start <service>` | Bật service |
| `systemctl stop <service>` | Tắt service |
| `systemctl restart <service>` | Tắt rồi bật lại |
| `systemctl enable <service>` | Tự động bật khi khởi động máy |
| `systemctl disable <service>` | Không tự bật |
| `systemctl is-active <service>` | Kiểm tra đang chạy không |

> 🎯 DevOps thực tế: `sudo systemctl restart nginx`, `sudo systemctl status docker`, `journalctl -u docker -f`.

## Thực hành từng bước

### Bước 1: Xem trạng thái SSH service
```bash
sudo systemctl status ssh
```

Output có thể là `active (running)` hoặc `inactive`.

Nhấn `q` để thoát.

### Bước 2: Kiểm tra nhiều service quan trọng
```bash
systemctl is-active ssh
systemctl is-active cron
```

Output: `active` hoặc `inactive`.

### Bước 3: Xem tất cả service đang chạy
```bash
systemctl list-units --type=service --state=running | head -20
```

### Bước 4: Thử tắt và bật lại cron
```bash
sudo systemctl stop cron
systemctl is-active cron
sudo systemctl start cron
systemctl is-active cron
```

### Bước 5: Xem log service bằng journalctl
```bash
sudo journalctl -u ssh --no-pager | tail -20
```

- `-u ssh` = unit (service) ssh
- `--no-pager` = hiện ra luôn, không cần nhấn phím
- `tail -20` = 20 dòng cuối

### Bước 6: Theo dõi log realtime
```bash
sudo journalctl -u cron -f
```

Lệnh này giữ màn hình, hiện log mới khi có.

**Thoát:** Nhấn `Ctrl + C`.

### Bước 7: Xem log từ hôm nay
```bash
sudo journalctl --since today --no-pager | tail -30
```

### Bước 8: Xem boot log
```bash
sudo journalctl --boot --no-pager | tail -20
```

## Task cuối bài (15 phút)

1. Kiểm tra `ssh` đang chạy không.
2. Nếu không chạy, hãy bật lên.
3. Cho phép `ssh` tự động chạy khi WSL khởi động (dùng `enable`).
4. Xem 10 dòng log cuối của `ssh`.
5. Tìm trong log xem có từ "Server" không (dùng `grep`).

## Đáp án / Gợi ý

```bash
# Task 1
systemctl is-active ssh

# Task 2
sudo systemctl start ssh

# Task 3
sudo systemctl enable ssh

# Task 4
sudo journalctl -u ssh --no-pager | tail -10

# Task 5
sudo journalctl -u ssh --no-pager | grep Server
```

## Lưu ý quan trọng
- `systemctl restart` khác `reload`: `restart` tắt rồi mở lại, `reload` chỉ nạp lại config (không mất kết nối).
- DevOps hay dùng: `systemctl restart docker`, `journalctl -u nginx -f`.
- WSL không có systemd đầy đủ như server thật (WSL dùng `init` đặc biệt). Một số service có thể không có hoặc hoạt động khác. Trên server Ubuntu thật, `systemctl` hoạt động hoàn hảo.

---

# Bài 9: Disk & Storage

> ⏱️ Thời gian: ~60 phút (15p lý thuyết + 30p thực hành + 15p task)

## Mục tiêu
- Kiểm tra dung lượng đĩa (`df`, `du`)
- Tìm file/thư mục chiếm nhiều dung lượng
- Hiểu mount point
- DevOps cần monitor disk để tránh server "đầy ổ"

## Lý thuyết

| Lệnh | Ý nghĩa |
|------|---------|
| `df -h` | Xem dung lượng ổ đĩa (human readable) |
| `du -sh <đường dẫn>` | Xem dung lượng 1 thư mục |
| `du -h --max-depth=1` | Xem dung lượng các thư mục con |
| `lsblk` | Xem cấu trúc ổ đĩa/block |
| `mount` | Xem mount point |

> 🎯 DevOps thực tế: Server đầy disk = ứng dụng crash. Cần alert khi disk > 80%.

## Thực hành từng bước

### Bước 1: Xem tổng quan ổ đĩa
```bash
df -h
```

Output ví dụ:
```
Filesystem      Size  Used Avail Use% Mounted on
/dev/sdb        251G   15G  224G   7% /
tmpfs           3.9G     0  3.9G   0% /run
```

Giải thích cột:
- `Size` = tổng dung lượng
- `Used` = đã dùng
- `Avail` = còn trống
- `Use%` = % đã dùng
- `Mounted on` = mount point (thư mục gốc `/`)

### Bước 2: Xem dung lượng thư mục home
```bash
du -sh ~
```

Output: `15G /home/hoang` (con số khác tùy máy).

### Bước 3: Xem chi tiết từng thư mục con trong home
```bash
du -h --max-depth=1 ~ | sort -h
```

> `sort -h` = sort theo đơn vị human (K, M, G).

### Bước 4: Tìm thư mục nào "nặng" nhất
```bash
du -h --max-depth=1 /var/log 2>/dev/null | sort -rh | head -5
```

> `2>/dev/null` = bỏ qua lỗi permission denied.

### Bước 5: Xem cấu trúc ổ đĩa
```bash
lsblk
```

### Bước 6: Tạo file lớn và quan sát
```bash
cd ~/hoc-linux/devops-roadmap
dd if=/dev/zero of=bigfile bs=1M count=100
```

- Tạo file 100MB toàn số 0.

Kiểm tra:
```bash
ls -lh bigfile
df -h .
```

Dọn dẹp:
```bash
rm bigfile
```

### Bước 7: Tìm file lớn nhất trong một thư mục
```bash
find ~ -type f -size +10M -exec ls -lh {} \; 2>/dev/null
```

> Tìm file > 10MB trong home.

## Task cuối bài (15 phút)

1. Xem % disk đã dùng của `/` (gốc).
2. Xem tổng dung lượng thư mục `/var/log`.
3. Tìm 3 file lớn nhất trong `/home/hoang` (dùng `find` + `ls` hoặc `du`).
4. Tạo file `test_50mb` kích thước 50MB, kiểm tra lại dung lượng, rồi xóa.

## Đáp án / Gợi ý

```bash
# Task 1
df -h /

# Task 2
sudo du -sh /var/log

# Task 3
find /home/hoang -type f -exec ls -lh {} \; 2>/dev/null | sort -k5 -rh | head -3
# Hoặc đơn giản hơn:
find /home/hoang -type f -size +1M -exec ls -lh {} \; 2>/dev/null

# Task 4
cd ~/hoc-linux/devops-roadmap
dd if=/dev/zero of=test_50mb bs=1M count=50
ls -lh test_50mb
rm test_50mb
```

## Lưu ý quan trọng
- `df -h` xem ổ đĩa, `du -sh` xem thư mục. Nhiều người nhầm lẫn.
- Server production cần monitor disk space. Công cụ như Prometheus + Alertmanager sẽ alert khi `Use% > 80`.
- Log file (`/var/log`) là thủ phạm thường gặp khi server đầy disk. DevOps dùng `logrotate` để quản lý (học ở Bài 13).

---

# Bài 10: Networking Sâu Hơn

> ⏱️ Thời gian: ~60 phút (15p lý thuyết + 30p thực hành + 15p task)

## Mục tiêu
- Xem IP, port, kết nối mạng
- Dùng `curl`, `wget` để gọi API/tải file
- Hiểu `/etc/hosts`, hostname
- DevOps cần kiểm tra kết nối container, server, load balancer

## Lý thuyết

| Lệnh | Ý nghĩa |
|------|---------|
| `ip a` hoặc `ifconfig` | Xem địa chỉ IP |
| `hostname` | Xem tên máy |
| `ping <host>` | Kiểm tra kết nối mạng |
| `curl <url>` | Gọi HTTP request |
| `wget <url>` | Tải file |
| `ss -tulpn` | Xem port đang mở |
| `netstat -tulpn` | Tương tự `ss` (cũ hơn) |

> 🎯 DevOps thực tế: `curl -I http://localhost:8080/health` kiểm tra app sống không, `ss -tulpn | grep 80` xem nginx đang nghe port nào.

## Thực hành từng bước

### Bước 1: Xem IP và network interface
```bash
ip a
```

Tìm dòng `inet` — đó là IP của bạn (thường `172.x.x.x` hoặc `192.168.x.x` trong WSL).

```bash
hostname
hostname -I
```

### Bước 2: Kiểm tra kết nối internet
```bash
ping -c 4 google.com
```

> `-c 4` = chỉ gửi 4 gói tin. Nhấn `Ctrl+C` nếu không có `-c`.

Output:
```
64 bytes from ...: icmp_seq=1 ttl=117 time=15.3 ms
```

### Bước 3: Gọi HTTP bằng curl
```bash
curl https://api.github.com
```

Output: JSON dài. Để chỉ xem header:
```bash
curl -I https://api.github.com
```

> `-I` = chỉ lấy header.

Output:
```
HTTP/2 200
server: GitHub.com
```

### Bước 4: Tải file bằng wget
```bash
cd ~/hoc-linux/devops-roadmap
wget https://raw.githubusercontent.com/docker/docker-ce/master/README.md -O docker_readme.md
ls -lh docker_readme.md
```

> `-O` = đặt tên file lưu.

### Bước 5: Xem port đang mở
```bash
ss -tulpn
```

Giải thích tùy chọn:
- `-t` = TCP
- `-u` = UDP
- `-l` = listening (đang chờ kết nối)
- `-p` = hiện process
- `-n` = hiện số, không tìm tên

Output ví dụ:
```
tcp   LISTEN  0  128  0.0.0.0:22  0.0.0.0:*  users:(("sshd",pid=123,fd=3))
```

Nghĩa là: có process `sshd` đang nghe port 22.

### Bước 6: Xem file hosts
```bash
cat /etc/hosts
```

Output:
```
127.0.0.1       localhost
127.0.1.1       DESKTOP-...
```

Bạn có thể thêm dòng để đổi tên miền local:
```bash
echo "127.0.0.1 myapp.local" | sudo tee -a /etc/hosts
cat /etc/hosts | tail -3
```

> `sudo tee -a` = ghi file với quyền root.

Giờ ping thử:
```bash
ping -c 2 myapp.local
```

### Bước 7: Kiểm tra DNS
```bash
nslookup google.com
dig google.com +short
```

> `dig` có thể chưa có, cài bằng `sudo apt install dnsutils -y`.

## Task cuối bài (15 phút)

1. Xem IP của máy bạn (`ip a`).
2. Ping `github.com` 3 lần.
3. Dùng `curl` gọi đến `https://api.github.com/users/github` và ghi output ra `github_user.json`.
4. Xem file `github_user.json` bằng `head`.
5. Kiểm tra port 22 (SSH) có đang mở không bằng `ss`.

## Đáp án / Gợi ý

```bash
# Task 1
ip a | grep inet

# Task 2
ping -c 3 github.com

# Task 3
curl https://api.github.com/users/github > github_user.json

# Task 4
head -20 github_user.json

# Task 5
ss -tulpn | grep :22
```

## Lưu ý quan trọng
- `ss` thay thế `netstat` trên Linux hiện đại. DevOps hay dùng: `ss -tulpn | grep 80`.
- `curl` mạnh hơn `wget` cho API testing. DevOps test health check: `curl -f http://localhost:8080/health || echo "DOWN"`.
- `/etc/hosts` dùng để test local trước khi đổi DNS thật.
- Container/docker sẽ expose port. Bạn dùng `ss` để xem port mapping.

---

# Bài 11: SSH & Remote Access

> ⏱️ Thời gian: ~60 phút (15p lý thuyết + 30p thực hành + 15p task)

## Mục tiêu
- Hiểu SSH là gì
- Tạo SSH key pair (public/private)
- Copy file bằng `scp`, `rsync`
- DevOps dùng SSH để quản lý hàng trăm server

## Lý thuyết

**SSH** (Secure Shell) = giao thức để kết nối an toàn vào server từ xa.

Thay vì đăng nhập bằng mật khẩu, DevOps dùng **SSH key** — an toàn hơn, tự động hóa được.

| File | Ý nghĩa |
|------|---------|
| `~/.ssh/id_rsa` | Private key (giữ bí mật, không chia sẻ) |
| `~/.ssh/id_rsa.pub` | Public key (copy lên server) |
| `~/.ssh/authorized_keys` | Danh sách public key được phép đăng nhập |

| Lệnh | Ý nghĩa |
|------|---------|
| `ssh-keygen` | Tạo key pair |
| `ssh user@server` | Đăng nhập vào server |
| `scp file user@server:/path` | Copy file lên server |
| `rsync -avz file user@server:/path` | Đồng bộ file (mạnh hơn scp) |

> 🎯 DevOps thực tế: Ansible dùng SSH key để quản lý 1000 server. CI/CD dùng SSH để deploy lên staging/production.

## Thực hành từng bước

### Bước 1: Kiểm tra SSH client
```bash
ssh -V
```

### Bước 2: Tạo SSH key pair
```bash
ssh-keygen -t rsa -b 4096 -C "hoang_devops"
```

Nhấn `Enter` 3 lần (chấp nhận đường dẫn mặc định, không đặt passphrase).

Output:
```
Your identification has been saved in /home/hoang/.ssh/id_rsa
Your public key has been saved in /home/hoang/.ssh/id_rsa.pub
```

### Bước 3: Xem public key
```bash
cat ~/.ssh/id_rsa.pub
```

Output dạng:
```
ssh-rsa AAAA... hoang_devops
```

> Nếu bạn có server thật, copy dòng này vào `~/.ssh/authorized_keys` trên server.

### Bước 4: Thử SSH vào chính mình (localhost)
```bash
ssh localhost
```

Nếu hỏi password → nhập password WSL của bạn. Nếu báo `Connection refused` → SSH chưa chạy:
```bash
sudo systemctl start ssh
ssh localhost
```

Thoát:
```bash
exit
```

### Bước 5: Copy public key vào localhost (tự động đăng nhập)
```bash
ssh-copy-id localhost
```

Nhập password 1 lần. Lần sau SSH sẽ không cần password:
```bash
ssh localhost
exit
```

### Bước 6: Copy file bằng scp
```bash
cd ~/hoc-linux/devops-roadmap
echo "Remote file content" > remote_test.txt
scp remote_test.txt localhost:/tmp/
```

Kiểm tra:
```bash
ssh localhost "cat /tmp/remote_test.txt"
```

### Bước 7: Đồng bộ bằng rsync
```bash
rsync -avz ~/hoc-linux/devops-roadmap/ localhost:/tmp/backup_devops/
```

- `-a` = archive (giữ quyền, timestamp)
- `-v` = verbose (hiện chi tiết)
- `-z` = compress (nén khi truyền)

Kiểm tra:
```bash
ssh localhost "ls -la /tmp/backup_devops/"
```

## Task cuối bài (15 phút)

1. Xem nội dung file `~/.ssh/id_rsa.pub`.
2. Tạo file `deploy.txt` và copy lên `/tmp/deploy.txt` trên localhost bằng `scp`.
3. Dùng `rsync` đồng bộ thư mục `~/hoc-linux` vào `/tmp/hoc_backup`.
4. SSH vào localhost và xóa `/tmp/deploy.txt` từ xa (dùng lệnh trong dấu ngoặc kép).

## Đáp án / Gợi ý

```bash
# Task 1
cat ~/.ssh/id_rsa.pub

# Task 2
echo "Deploy version 1.0" > deploy.txt
scp deploy.txt localhost:/tmp/deploy.txt

# Task 3
rsync -avz ~/hoc-linux/ localhost:/tmp/hoc_backup/

# Task 4
ssh localhost "rm /tmp/deploy.txt"
```

## Lưu ý quan trọng
- **Không bao giờ** chia sẻ `id_rsa` (private key). Chỉ chia sẻ `id_rsa.pub`.
- File `~/.ssh` phải có quyền `700`, `authorized_keys` phải có quyền `600`.
- `rsync` mạnh hơn `scp` vì chỉ copy phần thay đổi. DevOps dùng rsync để deploy code.
- Nếu SSH báo `WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED`, xóa dòng tương ứng trong `~/.ssh/known_hosts`.

---

# Bài 12: Cron & Automation

> ⏱️ Thời gian: ~60 phút (15p lý thuyết + 30p thực hành + 15p task)

## Mục tiêu
- Dùng `crontab` để lập lịch chạy lệnh tự động
- Hiểu cú pháp thời gian của cron
- Dùng `at` để chạy lệnh 1 lần
- DevOps dùng cron để backup, dọn log, health check định kỳ

## Lý thuyết

**Cron** = scheduler của Linux. Chạy lệnh theo thời gian định sẵn.

Cú pháp crontab:
```
* * * * * command
│ │ │ │ │
│ │ │ │ └── Thứ trong tuần (0-7, Chủ Nhật = 0 hoặc 7)
│ │ │ └──── Tháng (1-12)
│ │ └────── Ngày trong tháng (1-31)
│ └──────── Giờ (0-23)
└────────── Phút (0-59)
```

Ví dụ:
| Cron | Ý nghĩa |
|------|---------|
| `0 2 * * *` | 2:00 sáng mỗi ngày |
| `*/5 * * * *` | Mỗi 5 phút |
| `0 0 * * 0` | 0:00 Chủ Nhật hàng tuần |
| `@reboot` | Mỗi khi khởi động |

> 🎯 DevOps thực tế: Backup database 2h sáng, dọn log 3h sáng, health check mỗi 5 phút.

## Thực hành từng bước

### Bước 1: Kiểm tra cron có chạy không
```bash
systemctl is-active cron
```

Nếu không active:
```bash
sudo systemctl start cron
sudo systemctl enable cron
```

### Bước 2: Xem crontab hiện tại
```bash
crontab -l
```

Output: `no crontab for hoang` (nếu chưa có).

### Bước 3: Mở crontab để sửa
```bash
crontab -e
```

Lần đầu hỏi chọn editor → chọn `1` (nano) để dễ dùng.

### Bước 4: Thêm job đơn giản — ghi log mỗi phút
Thêm dòng sau vào cuối file:
```
* * * * * echo "Cron chay luc $(date)" >> /home/hoang/hoc-linux/devops-roadmap/cron_log.txt
```

Lưu: `Ctrl+O` → `Enter` → `Ctrl+X`.

### Bước 5: Kiểm tra job đã lưu
```bash
crontab -l | tail -3
```

### Bước 6: Đợi 1-2 phút rồi kiểm tra log
```bash
cd ~/hoc-linux/devops-roadmap
cat cron_log.txt
```

Output có thể như:
```
Cron chay luc Thu Jul 16 08:15:01 UTC 2026
Cron chay luc Thu Jul 16 08:16:01 UTC 2026
```

> 💡 Nếu không thấy gì sau 2 phút, kiểm tra lại đường dẫn trong crontab.

### Bước 7: Xóa job thử nghiệm
```bash
crontab -e
```

Xóa dòng vừa thêm → lưu → thoát.

### Bước 8: Tạo script backup tự động
```bash
cd ~/hoc-linux/devops-roadmap
cat > backup.sh << 'EOF'
#!/bin/bash
BACKUP_DIR="/home/hoang/hoc-linux/devops-roadmap/backup_$(date +%Y%m%d_%H%M%S)"
mkdir -p "$BACKUP_DIR"
cp -r /home/hoang/hoc-linux/devops-roadmap/*.txt "$BACKUP_DIR/" 2>/dev/null
echo "Backup done at $(date)" >> /home/hoang/hoc-linux/devops-roadmap/backup_history.txt
EOF
chmod +x backup.sh
./backup.sh
```

Kiểm tra:
```bash
ls backup_*
cat backup_history.txt
```

### Bước 9: Lên lịch chạy backup
```bash
crontab -e
```

Thêm:
```
0 2 * * * /home/hoang/hoc-linux/devops-roadmap/backup.sh
```

Nghĩa là: 2:00 sáng mỗi ngày chạy script.

> ⚠️ Để test ngay mà không đợi 2h sáng, bạn có thể thay `0 2` bằng `* *` (mỗi phút), test xong rồi sửa lại.

### Bước 10: Dùng `at` để chạy 1 lần
```bash
echo "echo 'Hello from at' >> ~/hoc-linux/devops-roadmap/at_log.txt" | at now + 1 minute
```

Kiểm tra danh sách:
```bash
atq
```

Sau 1 phút:
```bash
cat ~/hoc-linux/devops-roadmap/at_log.txt
```

## Task cuối bài (15 phút)

1. Tạo script `health_check.sh` ghi dòng `"Health OK - $(date)"` vào `health.log`.
2. Lập lịch chạy script đó mỗi 2 phút trong 10 phút (test), sau đó xóa job.
3. Kiểm tra `health.log` có ít nhất 3 dòng.
4. Dùng `at` để chạy lệnh `echo "Done"` sau 2 phút.

## Đáp án / Gợi ý

```bash
cd ~/hoc-linux/devops-roadmap

# Task 1
cat > health_check.sh << 'EOF'
#!/bin/bash
echo "Health OK - $(date)" >> /home/hoang/hoc-linux/devops-roadmap/health.log
EOF
chmod +x health_check.sh

# Task 2
crontab -e
# Thêm: */2 * * * * /home/hoang/hoc-linux/devops-roadmap/health_check.sh
# Đợi 10 phút, rồi xóa job bằng crontab -e

# Task 3
cat health.log | wc -l

# Task 4
echo 'echo "Done"' | at now + 2 minutes
```

## Lưu ý quan trọng
- Cron chạy với môi trường tối thiểu. Nên dùng **đường dẫn tuyệt đối** trong script (`/home/hoang/...` thay vì `~/...`).
- Output của cron job được gửi mail cho user. Trên server không có mail, nên luôn redirect output: `* * * * * /script.sh > /tmp/script.log 2>&1`.
- DevOps hay dùng `cron` + `logrotate` để tự động hóa hoàn toàn.
- WSL cron có thể không chạy khi WSL đang sleep. Trên server thật, cron luôn chạy 24/7.

---

# Bài 13: Log Management

> ⏱️ Thời gian: ~60 phút (15p lý thuyết + 30p thực hành + 15p task)

## Mục tiêu
- Tìm log trong `/var/log`
- Dùng `tail -f`, `journalctl` để theo dõi log realtime
- Hiểu `logrotate`
- DevOps phân tích log để debug sự cố

## Lý thuyết

Linux lưu log ở `/var/log/`:

| File/Thư mục | Chứa log của |
|--------------|--------------|
| `/var/log/syslog` hoặc `/var/log/messages` | Log hệ thống chung |
| `/var/log/auth.log` | Đăng nhập, xác thực |
| `/var/log/kern.log` | Kernel |
| `/var/log/dpkg.log` | Cài đặt package |
| `/var/log/nginx/` | Web server nginx |
| `/var/log/apache2/` | Web server Apache |

| Lệnh | Ý nghĩa |
|------|---------|
| `tail -f <file>` | Theo dõi cuối file realtime |
| `journalctl -u <service>` | Xem log của service |
| `grep "ERROR" <file>` | Tìm lỗi |
| `logrotate` | Tự động nén/xóa log cũ |

> 🎯 DevOps thực tế: `tail -f /var/log/nginx/access.log | grep 500` — xem lỗi server ngay lập tức.

## Thực hành từng bước

### Bước 1: Xem cấu trúc /var/log
```bash
ls -la /var/log/
```

### Bước 2: Xem log hệ thống (dùng sudo)
```bash
sudo tail -20 /var/log/syslog
```

Hoặc:
```bash
sudo tail -20 /var/log/messages 2>/dev/null || echo "Khong co messages, thu syslog"
```

### Bước 3: Theo dõi log realtime
```bash
sudo tail -f /var/log/syslog
```

Mở terminal thứ 2 (hoặc nhấn `Ctrl+Z`, rồi chạy lệnh khác, rồi `fg` để quay lại), hoặc chỉ quan sát.

**Thoát:** `Ctrl + C`.

### Bước 4: Tìm lỗi trong log
```bash
sudo grep -i "error" /var/log/syslog | tail -10
```

> `-i` = không phân biệt hoa thường.

### Bước 5: Xem log đăng nhập
```bash
sudo tail -10 /var/log/auth.log
```

Thử đăng nhập SSH vào localhost rồi xem lại:
```bash
ssh localhost
exit
sudo tail -5 /var/log/auth.log
```

### Bước 6: Theo dõi `company_app.log` realtime — Storyline

```bash
cd ~/hoc-linux/devops-roadmap
tail -f company_app.log &
```

> `&` = chạy background. `tail -f` theo dõi file log realtime — công cụ sống còn của DevOps khi debug.

Thêm log mới (giả lập ứng dụng đang chạy):
```bash
echo "$(date '+%Y-%m-%d %H:%M:%S') INFO User login success" >> company_app.log
echo "$(date '+%Y-%m-%d %H:%M:%S') INFO Database connected" >> company_app.log
```

Quay lại terminal đang chạy `tail -f` (nhấn `fg` nếu bạn đã nhấn `Ctrl+Z`), hoặc kill job background:
```bash
kill %1
```

### Bước 7: Hiểu logrotate với `company_app.log` — Storyline
Xem cấu hình:
```bash
cat /etc/logrotate.conf | head -20
```

Ví dụ cấu hình logrotate cho log dự án của bạn:
```bash
sudo tee /etc/logrotate.d/company_app << 'EOF'
/home/hoang/hoc-linux/devops-roadmap/company_app.log {
    daily
    rotate 7
    compress
    missingok
    notifempty
}
EOF
```

Giải thích:
- `daily` = xoay log mỗi ngày
- `rotate 7` = giữ 7 bản
- `compress` = nén file cũ
- `missingok` = không lỗi nếu file không có
- `notifempty` = không xoay nếu file trống

Thử chạy logrotate:
```bash
sudo logrotate -d /etc/logrotate.d/myapp
```

> `-d` = debug (không thực sự chạy, chỉ xem sẽ làm gì).

## Task cuối bài (15 phút)

1. **Mở rộng `company_app.log`** — thêm 5 dòng log mới (gồm đủ INFO, WARN, ERROR) bằng `>>`.
2. Đếm tổng số dòng ERROR trong `company_app.log`.
3. Dùng `tail -f` theo dõi `company_app.log` trong background, append thêm 3 dòng mới, rồi kill `tail`.
4. Tìm trong `/var/log` xem file nào được sửa đổi gần nhất (`ls -lt /var/log | head`).
5. **(Storyline)** Kiểm tra `company_app.log` có bao nhiêu dòng tổng cộng (`wc -l`).

## Đáp án / Gợi ý

```bash
cd ~/hoc-linux/devops-roadmap

# Task 1 — Thêm dòng vào company_app.log
cat >> company_app.log << 'EOF'
2025-07-10 INFO Payment processed
2025-07-10 WARN Response slow
2025-07-10 ERROR Cache miss
2025-07-10 INFO Email sent
2025-07-10 ERROR Queue full
EOF

# Task 2
grep ERROR company_app.log | wc -l

# Task 3
tail -f company_app.log &
echo "$(date '+%Y-%m-%d %H:%M:%S') INFO New event 1" >> company_app.log
echo "$(date '+%Y-%m-%d %H:%M:%S') INFO New event 2" >> company_app.log
echo "$(date '+%Y-%m-%d %H:%M:%S') WARN Disk warning" >> company_app.log
kill %1

# Task 4
ls -lt /var/log | head -10

# Task 5
wc -l company_app.log
```

## Lưu ý quan trọng
- `tail -f` là công cụ sống còn của DevOps khi debug production incident.
- Log file lớn có thể làm đầy disk. `logrotate` giải quyết vấn đề này.
- `journalctl` trên systemd hiện đại thay thế nhiều file log cũ.
- Trong Docker, log thường ra stdout. DevOps dùng `docker logs -f container`.

---

# Bài 14: File Ownership & Advanced Find

> ⏱️ Thời gian: ~60 phút (15p lý thuyết + 30p thực hành + 15p task)

## Mục tiêu
- Dùng `chown`, `chgrp` thay đổi chủ sở hữu file
- Dùng `find` nâng cao: tìm theo size, thời gian, thực thi lệnh trên kết quả
- Hiểu tại sao DevOps hay gặp lỗi permission và cách sửa

## Lý thuyết

| Lệnh | Ý nghĩa |
|------|---------|
| `chown user:group file` | Đổi owner và group |
| `chown user file` | Chỉ đổi owner |
| `chgrp group file` | Chỉ đổi group |
| `find . -name "*.log"` | Tìm file theo tên |
| `find . -size +10M` | Tìm file > 10MB |
| `find . -mtime +7` | Tìm file sửa > 7 ngày trước |
| `find . -exec command {} \;` | Chạy lệnh trên mỗi file tìm được |

> 🎯 DevOps thực tế: `find /var/log -name "*.log" -mtime +30 -delete` — xóa log cũ hơn 30 ngày.

## Thực hành từng bước

### Bước 1: Tạo file và xem owner
```bash
cd ~/hoc-linux/devops-roadmap
touch owner_test.txt
ls -l owner_test.txt
```

Output: `hoang hoang` (owner + group).

### Bước 2: Tạo group mới và đổi group của file
```bash
sudo groupadd devops
sudo chgrp devops owner_test.txt
ls -l owner_test.txt
```

Output: `hoang devops`.

### Bước 3: Tạo user mới và đổi owner
```bash
sudo useradd -m tester
sudo chown tester owner_test.txt
ls -l owner_test.txt
```

Output: `tester devops`.

### Bước 4: Đổi cả owner và group một lúc
```bash
sudo chown hoang:hoang owner_test.txt
ls -l owner_test.txt
```

Output: `hoang hoang`.

### Bước 5: Đổi owner cả thư mục đệ quy
```bash
mkdir -p test_chown/sub1/sub2
touch test_chown/file1.txt test_chown/sub1/file2.txt
sudo chown -R tester:tester test_chown
ls -la test_chown
ls -la test_chown/sub1
```

> `-R` = recursive (đệ quy).

### Bước 6: Tìm file theo size
```bash
find ~ -type f -size +1M -ls 2>/dev/null | head -10
```

> `-ls` = hiện chi tiết như `ls -dils`.

### Bước 7: Tìm file sửa đổi trong 24h qua
```bash
find ~/hoc-linux/devops-roadmap -type f -mtime -1
```

### Bước 8: Tìm và xóa file `.tmp` cũ hơn 7 ngày
```bash
cd ~/hoc-linux/devops-roadmap
touch old1.tmp
touch old2.tmp
find . -name "*.tmp" -mtime +7
```

Chưa thấy vì vừa tạo. Giả lập bằng cách touch với ngày cũ:
```bash
touch -d "10 days ago" old1.tmp
find . -name "*.tmp" -mtime +7
```

Output: `./old1.tmp`

Xóa:
```bash
find . -name "*.tmp" -mtime +7 -delete
```

### Bước 9: Tìm và chạy lệnh trên kết quả
```bash
find . -name "*.txt" -exec wc -l {} \;
```

> `{}` = thay thế bằng tên file tìm được. `\;` = kết thúc lệnh exec.

### Bước 10: Kiểm tra & đổi quyền `company_app.log` — Storyline

```bash
cd ~/hoc-linux/devops-roadmap
ls -l company_app.log
```

Kiểm tra ai là owner. Sau đó thử đổi group:
```bash
sudo chgrp devops company_app.log
ls -l company_app.log
```

Tìm tất cả file log trong thư mục:
```bash
find . -name "*.log"
```

> 🎯 **Storyline:** Bạn vừa thực hành quản lý quyền trên file log thực tế. Trong production, log file thường thuộc về user chạy service (ví dụ: `www-data`, `appuser`).

## Task cuối bài (15 phút)

1. Tạo 5 file: `data1.txt` đến `data5.txt`. Đặt owner là `root`, group là `devops`.
2. Tìm tất cả file `.txt` trong thư mục hiện tại và in tên file.
3. Tìm file `.txt` nào trống (size 0) và xóa chúng.
4. Dùng `find` để đếm tổng số dòng của tất cả file `.txt` trong thư mục hiện tại.
5. **(Storyline)** Kiểm tra `company_app.log` có đang nằm trong thư mục không (`find`), và xem dung lượng của nó (`du -sh`).

## Đáp án / Gợi ý

```bash
cd ~/hoc-linux/devops-roadmap

# Task 1
for i in 1 2 3 4 5; do touch data${i}.txt; done
sudo chown root:devops data*.txt
ls -l data*.txt

# Task 2
find . -maxdepth 1 -name "*.txt"

# Task 3
find . -maxdepth 1 -name "*.txt" -size 0 -delete
# Hoặc trước khi xóa, kiểm tra:
find . -maxdepth 1 -name "*.txt" -size 0

# Task 4
find . -maxdepth 1 -name "*.txt" -exec cat {} \; | wc -l

# Task 5
find . -name "company_app.log"
du -sh company_app.log
```

## Lưu ý quan trọng
- Lỗi permission là nguyên nhân hàng đầu khi deploy thất bại. Luôn kiểm tra `ls -la`.
- `chown -R` rất nguy hiểm. Đừng chạy `sudo chown -R user /` — sẽ phá hủy hệ thống.
- `find -exec` chậm hơn `find ... | xargs ...` khi số lượng file lớn. DevOps hay dùng: `find . -name "*.log" | xargs rm -f`.
- `find` kết hợp `-mtime`, `-size`, `-name` là công cụ dọn dẹp server tự động.

---

## ✅ Kết Thúc Giai Đoạn 2

Sau 8 bài, bạn đã biết:
- Quản lý user & sudo
- Quản lý service (systemctl)
- Quản lý disk & log
- Network, SSH, cron automation
- Tìm kiếm nâng cao

**Bạn giờ có thể:**
- SSH vào server, check service, xem log
- Lập lịch backup, dọn dẹp file cũ
- Copy deploy artifact lên server

Sẵn sàng thực chiến? Mở file `03-giai-doan-3-thuc-chien-devops.md`! 🚀
