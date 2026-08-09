# Giai Đoạn 1: Linux Core

> 6 bài học | Mỗi bài ~60 phút | Mục tiêu: Tự tin dùng Linux cơ bản

---

# Bài 1: Permissions (Quyền Truy Cập File)

> ⏱️ Thời gian: ~60 phút (15p lý thuyết + 30p thực hành + 15p task)

## Mục tiêu
- Hiểu `rwx` (read/write/execute) là gì
- Đọc được chuỗi quyền như `-rw-r--r--`
- Dùng `chmod` và `chown` để thay đổi quyền
- Biết tại sao DevOps cần `chmod +x` cho script deploy

## Lý thuyết

Linux quản lý quyền theo 3 nhóm:
| Nhóm | Ký hiệu | Ý nghĩa |
|------|---------|---------|
| Owner (người tạo) | `u` | Quyền của bạn |
| Group (nhóm) | `g` | Quyền của nhóm bạn thuộc về |
| Others (người khác) | `o` | Quyền của tất cả user khác trên máy |

3 loại quyền:
| Quyền | Ký hiệu | Số | Ý nghĩa |
|-------|---------|----|---------|
| Read | `r` | 4 | Đọc file |
| Write | `w` | 2 | Sửa file |
| Execute | `x` | 1 | Chạy file (nếu là script/chương trình) |

Cách đọc `ls -l`:
```
-rw-r--r-- 1 hoang hoang  123 Jan 1 10:00 file.txt
 │  │  │
 │  │  └── others: r-- (đọc)
 │  └───── group: r-- (đọc)
 └──────── owner: rw- (đọc + sửa)
```

Công thức số:
- `7` = `4+2+1` = `rwx`
- `6` = `4+2+0` = `rw-`
- `5` = `4+0+1` = `r-x`
- `4` = `4+0+0` = `r--`

## Thực hành từng bước

### Bước 1: Vào thư mục học tập
```bash
cd ~/hoc-linux/devops-roadmap
pwd
```

### Bước 2: Tạo file thử nghiệm
```bash
echo "Hello Permissions" > test_perm.txt
ls -l test_perm.txt
```

**Output mong đợi:**
```
-rw-r--r-- 1 hoang hoang 18 Jul 16 07:30 test_perm.txt
```

Giải thích: Bạn (hoang) có quyền đọc+sửa, nhóm và người khác chỉ đọc.

### Bước 3: Thử đổi quyền bằng số
```bash
chmod 777 test_perm.txt
ls -l test_perm.txt
```

**Output mong đợi:**
```
-rwxrwxrwx 1 hoang hoang 18 Jul 16 07:30 test_perm.txt
```

> ⚠️ `777` nghĩa là ai cũng có quyền đọc/sửa/chạy. **Không dùng trên server thật!**

### Bước 4: Đặt quyền an toàn cho file văn bản
```bash
chmod 644 test_perm.txt
ls -l test_perm.txt
```

**Output:**
```
-rw-r--r-- 1 hoang hoang 18 Jul 16 07:30 test_perm.txt
```

Giải thích: `644` = owner đọc+sửa (6), còn lại chỉ đọc (4,4).

### Bước 5: Tạo script và cho phép chạy (rất quan trọng với DevOps)
```bash
echo '#!/bin/bash' > deploy.sh
echo 'echo "Dang deploy..."' >> deploy.sh
ls -l deploy.sh
```

Output: `-rw-r--r--` → chưa chạy được!

Thử chạy:
```bash
./deploy.sh
```

**Output (lỗi):**
```
bash: ./deploy.sh: Permission denied
```

Thêm quyền chạy:
```bash
chmod +x deploy.sh
ls -l deploy.sh
./deploy.sh
```

**Output mong đợi:**
```
-rwxr-xr-x 1 hoang hoang ... deploy.sh
Dang deploy...
```

> 🎯 **DevOps thực tế:** Mọi file script deploy (`deploy.sh`, `setup.sh`, `install.sh`) đều cần `chmod +x` trước khi chạy.
> 
> 📌 **Lưu ý:** File `deploy.sh` này sẽ được **tái sử dụng xuyên suốt roadmap**. Bạn sẽ sửa nó ở Bài 4 (thêm biến môi trường), Bài 6 (bằng vim), và nâng cấp thành script thực tế ở Bài 15, 17.

### Bước 6: Đổi quyền theo ký tự (cách khác)
```bash
chmod u=rwx,g=rx,o=r deploy.sh
ls -l deploy.sh
```

Kết quả giống trên.

### Bước 7: Tạo thư mục và quyền thư mục
```bash
mkdir test_dir
ls -ld test_dir
```

Output: `drwxr-xr-x` → `d` ở đầu nghĩa là directory (thư mục).

```bash
chmod 700 test_dir
ls -ld test_dir
```

Output: `drwx------` → chỉ owner được vào, đọc, sửa.

## Task cuối bài (15 phút)

1. Tạo file `backup_script.sh` chứa nội dung:
   ```bash
   #!/bin/bash
   echo "Backup bat dau"
   echo "Backup thanh cong"
   ```
2. Đặt quyền: owner đọc+sửa+chạy, group và others chỉ đọc.
3. Chạy file và xem output.
4. Tạo thư mục `private_data` chỉ cho phép owner truy cập.

## Đáp án / Gợi ý

```bash
cd ~/hoc-linux/devops-roadmap

# Task 1
cat > backup_script.sh << 'EOF'
#!/bin/bash
echo "Backup bat dau"
echo "Backup thanh cong"
EOF

# Task 2
chmod 755 backup_script.sh
# Hoặc: chmod u=rwx,g=r,o=r backup_script.sh

# Task 3
./backup_script.sh

# Task 4
mkdir private_data
chmod 700 private_data
ls -ld private_data
```

## Lưu ý quan trọng
- `chmod 777` chỉ dùng khi test cục bộ, **không bao giờ** dùng trên server production.
- `chmod +x` là lệnh DevOps dùng nhiều nhất cho các file `.sh`.
- Nếu bạn copy file từ Windows vào WSL, file có thể mất quyền execute. Nhớ `chmod +x` lại.

---

# Bài 2: Process Management (Quản Lý Tiến Trình)

> ⏱️ Thời gian: ~60 phút (15p lý thuyết + 30p thực hành + 15p task)

## Mục tiêu
- Xem process đang chạy bằng `ps`, `top`
- Dừng process bằng `kill`
- Chạy process nền/foreground
- Biết DevOps cần monitor process (ví dụ: Docker daemon, nginx)

## Lý thuyết

**Process** = chương trình đang chạy trong bộ nhớ.

Mỗi process có **PID** (Process ID) — số định danh.

Trạng thái process:
- **Running** — đang chạy
- **Sleeping** — đang chờ (thường gặp)
- **Stopped** — bị tạm dừng (Ctrl+Z)
- **Zombie** — process chết nhưng chưa được dọn (hiếm gặp)

## Thực hành từng bước

### Bước 1: Xem process của bạn
```bash
ps
```

**Output mong đợi:**
```
  PID TTY          TIME CMD
  123 pts/0    00:00:00 bash
  456 pts/0    00:00:00 ps
```

### Bước 2: Xem TẤT CẢ process trên hệ thống
```bash
ps aux
```

Output rất dài. Để dừng xem, nhấn `q` (nếu đang trong `less`) hoặc lệnh tự chạy xong.

Lọc chỉ xem process liên quan đến bash:
```bash
ps aux | grep bash
```

> 🎯 Pipe `|` — đưa output của `ps aux` vào `grep`. DevOps dùng liên tục để tìm process.

### Bước 3: Tìm PID của một process cụ thể
```bash
pgrep bash
```

Output: một dãy số (PID).

### Bước 4: Giám sát realtime với `top`
```bash
top
```

Màn hình hiện ra:
- Dòng đầu: thời gian, uptime, số user
- `%Cpu(s)` — CPU đang dùng bao nhiêu
- `KiB Mem` — bộ nhớ
- Danh sách process đang chạy

**Cách thoát `top`:** Nhấn `q`.

### Bước 5: Cài `htop` (đẹp hơn, dễ dùng hơn)
```bash
sudo apt update
sudo apt install htop -y
htop
```

**Cách thoát `htop`:** Nhấn `q` hoặc `F10`.

> 💡 Nếu chưa biết `sudo` — đơn giản là "chạy với quyền admin". Sẽ học kỹ ở Bài 7.

### Bước 6: Chạy một process và dừng nó
```bash
sleep 100
```

Lệnh này đợi 100 giây. Nó đang chạy ở foreground.

**Dừng ngay:** Nhấn `Ctrl+C`. Process bị kill.

### Bước 7: Tạm dừng process
```bash
sleep 200
```

Nhấn `Ctrl+Z`.

Output:
```
^Z
[1]+  Stopped                 sleep 200
```

Process chuyển sang trạng thái **Stopped**.

Kiểm tra:
```bash
jobs
```

Output:
```
[1]+  Stopped                 sleep 200
```

### Bước 8: Đưa stopped process chạy lại
Chạy nền (background):
```bash
bg %1
```

Kiểm tra lại:
```bash
jobs
```

Output:
```
[1]+  Running                 sleep 200 &
```

Dừng process này bằng kill:
```bash
kill %1
jobs
```

Output:
```
[1]+  Terminated              sleep 200
```

### Bước 9: Kill process bằng PID
```bash
sleep 300 &
```

Output:
```
[1] 12345
```

`12345` chính là PID.

```bash
kill 12345
```

> ⚠️ Nếu process "cứng đầu" không chịu chết:
> ```bash
> kill -9 12345
> ```
> `-9` = SIGKILL = buộc dừng ngay lập tức. Dùng khi cần thiết.

## Task cuối bài (15 phút)

1. Chạy `sleep 500` ở background ngay từ đầu (dùng `&`).
2. Tìm PID của process đó bằng `pgrep sleep`.
3. Dùng `kill` để dừng process.
4. Chạy `top`, tìm process `bash` trong danh sách, nhấn `q` để thoát.

## Đáp án / Gợi ý

```bash
# Task 1
sleep 500 &
# Ghi nhớ PID hiện ra

# Task 2
pgrep sleep

# Task 3
kill <PID>
# Hoặc kill -9 <PID> nếu không dừng

# Task 4
top
# Nhấn q để thoát
```

## Lưu ý quan trọng
- `Ctrl+C` = gửi tín hiệu ngắt (terminate), process có thể bắt và dọn dẹp trước khi chết.
- `Ctrl+Z` = tạm dừng, không kill.
- `kill -9` = giết ngay, không cho process dọn dẹp. Chỉ dùng khi bình thường không kill được.
- DevOps hay dùng: `ps aux | grep nginx`, `pgrep dockerd`, `kill -HUP <PID>` (reload config).

---

# Bài 3: Package Management (Cài Đặt Phần Mềm)

> ⏱️ Thời gian: ~60 phút (15p lý thuyết + 30p thực hành + 15p task)

## Mục tiêu
- Cài, gỡ, cập nhật phần mềm bằng `apt`
- Tìm kiếm package
- Hiểu repository là gì
- DevOps cần cài tool: docker, nginx, curl, vim, v.v.

## Lý thuyết

Ubuntu dùng **APT** (Advanced Package Tool) để quản lý phần mềm.

Các lệnh chính:
| Lệnh | Ý nghĩa |
|------|---------|
| `apt update` | Cập nhật danh sách package từ internet |
| `apt upgrade` | Nâng cấp package đã cài lên bản mới |
| `apt install <tên>` | Cài package |
| `apt remove <tên>` | Gỡ package (giữ config) |
| `apt purge <tên>` | Gỡ package + xóa config |
| `apt search <từ>` | Tìm package |
| `apt list --installed` | Liệt kê package đã cài |
| `dpkg -l` | Liệt kê chi tiết (hệ thống cũ) |

> 🎯 DevOps thực tế: Mỗi khi setup server mới, bạn chạy `apt update && apt install -y docker.io nginx curl vim`.

## Thực hành từng bước

### Bước 1: Cập nhật danh sách package
```bash
sudo apt update
```

Output dài, kết thúc bằng:
```
Reading package lists... Done
```

### Bước 2: Cài đặt tree (tiện ích xem cây thư mục)
```bash
sudo apt install tree -y
```

> `-y` = tự động trả lời "Yes", không hỏi lại.

Kiểm tra:
```bash
tree --version
```

### Bước 3: Dùng tree để xem cấu trúc thư mục bài học
```bash
cd ~
tree -L 2
```

Output sẽ hiện cây thư mục đẹp mắt.

### Bước 4: Tìm package liên quan đến python
```bash
apt search python3 | head -20
```

> `head -20` = chỉ lấy 20 dòng đầu, không bị tràn màn hình.

### Bước 5: Xem package nào đã cài
```bash
apt list --installed | grep vim
```

### Bước 6: Gỡ cài đặt tree
```bash
sudo apt remove tree -y
tree
```

Output (lỗi):
```
Command 'tree' not found
```

Cài lại nếu muốn:
```bash
sudo apt install tree -y
```

### Bước 7: Nâng cấp hệ thống (không bắt buộc, có thể lâu)
```bash
sudo apt upgrade -y
```

> ⚠️ Có thể tốn vài phút tùy mạng. Nếu thấy lâu, nhấn `Ctrl+C` để hủy.

### Bước 8: Dọn dẹp package thừa
```bash
sudo apt autoremove -y
sudo apt autoclean
```

## Task cuối bài (15 phút)

1. Cài `curl` và `wget` nếu chưa có (thường đã có sẵn).
2. Kiểm tra version của `curl`.
3. Tìm package có tên chứa `net-tools`.
4. Cài `net-tools` (chứa lệnh `ifconfig`, `netstat`).
5. Chạy `ifconfig` hoặc `ip a` để xem IP.

## Đáp án / Gợi ý

```bash
# Task 1
curl --version || sudo apt install curl -y
wget --version || sudo apt install wget -y

# Task 2
curl --version

# Task 3
apt search net-tools

# Task 4
sudo apt install net-tools -y

# Task 5
ifconfig
# Hoặc nếu không có:
ip a
```

## Lưu ý quan trọng
- Luôn chạy `apt update` trước khi cài phần mềm mới.
- `-y` giúp script tự động chạy không cần người bấm Enter.
- DevOps hay viết Dockerfile/Cloud-init dùng `apt-get` (dùng trong script tự động, không có progress bar đẹp như `apt`).
- Đừng chạy `apt upgrade` trên server production giờ cao điểm, có thể restart service.

---

# Bài 4: Environment Variables & Alias

> ⏱️ Thời gian: ~60 phút (15p lý thuyết + 30p thực hành + 15p task)

## Mục tiêu
- Hiểu biến môi trường (`PATH`, `HOME`, `USER`)
- Tạo biến tùy chỉnh
- Tạo alias (lệnh tắt)
- Sửa file `.bashrc` để lưu thiết lập lâu dài

## Lý thuyết

**Biến môi trường** là các giá trị hệ thống lưu sẵn, các chương trình có thể đọc.

Biến quan trọng nhất:
| Biến | Ý nghĩa | Ví dụ |
|------|---------|-------|
| `PATH` | Danh sách thư mục chứa lệnh | `/usr/bin:/bin` |
| `HOME` | Thư mục home của user | `/home/hoang` |
| `USER` | Tên user hiện tại | `hoang` |
| `SHELL` | Shell đang dùng | `/bin/bash` |

**Alias** = đặt tên ngắn cho lệnh dài.

> 🎯 DevOps thực tế: CI/CD pipeline đọc biến `DOCKER_IMAGE`, `ENV=production` để quyết định deploy đâu.

## Thực hành từng bước

### Bước 1: Xem các biến môi trường
```bash
env | head -20
```

Hoặc xem từng biến:
```bash
echo $HOME
echo $USER
echo $PATH
```

Output:
```
/home/hoang
hoang
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

### Bước 2: PATH hoạt động thế nào
Khi bạn gõ `ls`, Linux tìm trong các thư mục trong `PATH`.

```bash
which ls
which bash
```

Output:
```
/usr/bin/ls
/usr/bin/bash
```

### Bước 3: Tạo biến tạm thời
```bash
export PROJECT_NAME="devops-demo"
echo $PROJECT_NAME
```

Output: `devops-demo`

Thử mở terminal mới (đóng WSL, mở lại):
```bash
echo $PROJECT_NAME
```

Output trống! Vì `export` chỉ có hiệu lực trong session hiện tại.

### Bước 4: Lưu biến lâu dài trong `.bashrc`
```bash
echo 'export PROJECT_NAME="devops-demo"' >> ~/.bashrc
cat ~/.bashrc | tail -5
```

Áp dụng thay đổi mà không cần đóng terminal:
```bash
source ~/.bashrc
echo $PROJECT_NAME
```

### Bước 5: Tạo alias
```bash
alias ll='ls -lah'
ll
```

Output: liệt kê file chi tiết (giống `ls -lah`).

Alias khác hữu ích:
```bash
alias ..='cd ..'
alias ...='cd ../..'
alias c='clear'
alias update='sudo apt update && sudo apt upgrade -y'
```

Thử:
```bash
c
```

Màn hình được xóa sạch.

### Bước 6: Lưu alias lâu dài
```bash
echo "alias ll='ls -lah'" >> ~/.bashrc
echo "alias c='clear'" >> ~/.bashrc
source ~/.bashrc
```

### Bước 7: Xem tất cả alias đang có
```bash
alias
```

### Bước 8: Xóa alias tạm thời
```bash
unalias c
c
```

Output (lỗi):
```
c: command not found
```

> Không sao, vì `c` không được lưu trong `.bashrc` (nếu bạn chưa lưu). Nếu đã lưu, chạy `source ~/.bashrc` lại.

### Bước 9: Sửa `deploy.sh` để đọc biến môi trường (Storyline)

Nhớ file `deploy.sh` đã tạo ở Bài 1? Giờ ta sẽ cập nhật nó để sử dụng biến môi trường.

```bash
cd ~/hoc-linux/devops-roadmap
cat deploy.sh
```

Output hiện tại:
```
#!/bin/bash
echo "Dang deploy..."
```

Ghi đè nội dung mới:
```bash
cat > deploy.sh << 'EOF'
#!/bin/bash
ENV="${ENV:-development}"
echo "Dang deploy..."
echo "Moi truong: $ENV"
EOF
chmod +x deploy.sh
```

Thử chạy với giá trị mặc định:
```bash
./deploy.sh
```

Output:
```
Dang deploy...
Moi truong: development
```

Thử chạy với giá trị tùy chỉnh:
```bash
ENV=production ./deploy.sh
```

Output:
```
Dang deploy...
Moi truong: production
```

> 🎯 **Storyline:** File `deploy.sh` giờ đã "lớn" hơn — biết đọc biến môi trường. Bạn sẽ tiếp tục sửa nó ở Bài 6 (bằng vim) và nâng cấp thành script thực tế ở Giai đoạn 3.

## Task cuối bài (15 phút)

1. Tạo biến `ENV=production` và lưu vào `.bashrc`.
2. Tạo alias `status='systemctl status'` (sẽ học `systemctl` ở Bài 8, nhưng cứ tạo alias trước).
3. Tạo alias `myip='ip a | grep inet'`.
4. Chạy `source ~/.bashrc` rồi thử `myip`.
5. Xem nội dung cuối file `.bashrc` bằng `tail -10 ~/.bashrc`.
6. **(Storyline)** Kiểm tra lại `deploy.sh` bằng `cat deploy.sh` để chắc nó đã được cập nhật.

## Đáp án / Gợi ý

```bash
# Task 1
echo 'export ENV=production' >> ~/.bashrc

# Task 2
echo "alias status='systemctl status'" >> ~/.bashrc

# Task 3
echo "alias myip='ip a | grep inet'" >> ~/.bashrc

# Task 4
source ~/.bashrc
myip

# Task 5
tail -10 ~/.bashrc

# Task 6
cat ~/hoc-linux/devops-roadmap/deploy.sh
```

## Lưu ý quan trọng
- `~/.bashrc` chạy mỗi khi mở terminal mới. Đây là nơi DevOps để alias, biến môi trường, custom prompt.
- `source ~/.bashrc` = chạy lại file mà không cần mở terminal mới.
- Đừng sửa đầu file `.bashrc` — thêm vào cuối file để dễ quản lý.
- Trong Docker/container, biến môi trường thường set bằng `ENV` trong Dockerfile hoặc `-e` khi `docker run`.

---

# Bài 5: Pipe & Redirect (Cốt Lõi Của DevOps)

> ⏱️ Thời gian: ~60 phút (15p lý thuyết + 30p thực hành + 15p task)

## Mục tiêu
- Dùng `|` (pipe) nối lệnh
- Dùng `>`, `>>`, `2>`, `&>` để ghi output/error ra file
- Kết hợp nhiều lệnh xử lý log (ví dụ: đếm số lỗi)
- Đây là kỹ năng **quan trọng nhất** cho DevOps

## Lý thuyết

**Pipe `|`**: Lấy output của lệnh bên trái, đưa vào input lệnh bên phải.

**Redirect**:
| Ký hiệu | Ý nghĩa |
|---------|---------|
| `>` | Ghi output đè lên file (overwrite) |
| `>>` | Ghi output thêm vào file (append) |
| `2>` | Ghi lỗi (stderr) ra file |
| `&>` | Ghi cả output và lỗi ra file |
| `<` | Lấy nội dung file làm input cho lệnh |

> 🎯 DevOps thực tế: `kubectl get pods | grep Error`, `docker logs container 2> errors.log`, `cat access.log | awk '{print $1}' | sort | uniq -c`.

## Thực hành từng bước

### Bước 1: Tạo file log mẫu (`company_app.log`) — Storyline

```bash
cd ~/hoc-linux/devops-roadmap
cat > company_app.log << 'EOF'
2025-07-10 INFO Server started
2025-07-10 ERROR Database timeout
2025-07-10 INFO Request processed
2025-07-10 ERROR Connection refused
2025-07-10 WARN Memory high
2025-07-10 ERROR Disk full
EOF
```

> 📌 **Storyline:** File `company_app.log` đại diện cho log của một ứng dụng thật. Bạn sẽ tiếp tục "chăm sóc" nó ở Bài 6 (sửa bằng vim), Bài 13 (theo dõi + logrotate), Bài 14 (tìm & đổi quyền), Bài 15 (viết script phân tích), và Bài 18 (audit).

### Bước 2: Pipe cơ bản — tìm dòng chứa ERROR
```bash
cat company_app.log | grep "ERROR"
```

Output:
```
2025-07-10 ERROR Database timeout
2025-07-10 ERROR Connection refused
2025-07-10 ERROR Disk full
```

> 💡 `cat company_app.log | grep ...` tương đương `grep "ERROR" company_app.log`. Pipe mạnh khi nối 3+ lệnh.

### Bước 3: Pipe nhiều tầng — đếm số lỗi
```bash
cat company_app.log | grep "ERROR" | wc -l
```

Output: `3`

Giải thích:
1. `cat company_app.log` → đọc file
2. `grep "ERROR"` → chỉ giữ dòng có ERROR
3. `wc -l` → đếm số dòng

### Bước 4: Tìm loại log duy nhất
```bash
cat company_app.log | awk '{print $3}' | sort | uniq
```

Output:
```
Connection
Database
Disk
Memory
Request
Server
```

> `awk '{print $3}'` = in cột thứ 3 (theo khoảng trắng).

### Bước 5: Đếm số lần xuất hiện mỗi loại log
```bash
cat company_app.log | awk '{print $3}' | sort | uniq -c
```

Output:
```
  1 Connection
  1 Database
  1 Disk
  1 Memory
  1 Request
  1 Server
```

### Bước 6: Redirect output ra file
```bash
cat company_app.log | grep "ERROR" > errors_only.txt
cat errors_only.txt
```

Output: chỉ có 3 dòng ERROR.

### Bước 7: Append thêm vào file
```bash
echo "2025-07-11 ERROR New error" >> errors_only.txt
cat errors_only.txt
```

Output: 4 dòng (3 cũ + 1 mới).

### Bước 8: Redirect lỗi ra file riêng
```bash
ls /thu_muc_khong_ton_tai 2> error.log
cat error.log
```

Output:
```
ls: cannot access '/thu_muc_khong_ton_tai': No such file or directory
```

### Bước 9: Ghi cả output lẫn lỗi
```bash
ls /home/hoang /khong_co &> all_output.txt
cat all_output.txt
```

### Bước 10: Chống lỗi — đưa lỗi vào "hố đen"
```bash
ls /khong_co 2> /dev/null
```

Không có output nào. `/dev/null` là "thùng rác vĩnh viễn" của Linux.

## Task cuối bài (15 phút)

1. **Mở rộng `company_app.log`** — thêm 10 dòng mới (gồm INFO, WARN, ERROR) bằng cách append (`>>`).
2. Tạo file `info.log` chỉ chứa dòng INFO từ `company_app.log`.
3. Tạo file `warn.log` chỉ chứa dòng WARN từ `company_app.log`.
4. Đếm tổng số dòng lỗi (WARN + ERROR) và ghi số đó vào `total_errors.txt`.
5. Dùng `sort` và `uniq -c` để đếm số dòng theo từng mức độ (INFO, WARN, ERROR) trong `company_app.log`.

## Đáp án / Gợi ý

```bash
cd ~/hoc-linux/devops-roadmap

# Task 1 — Thêm dòng vào company_app.log (dùng >> để giữ dòng cũ)
cat >> company_app.log << 'EOF'
2025-07-10 INFO Start
2025-07-10 INFO Connect DB
2025-07-10 WARN Slow query
2025-07-10 ERROR Timeout
2025-07-10 INFO Request 1
2025-07-10 INFO Request 2
2025-07-10 WARN High CPU
2025-07-10 ERROR DB down
2025-07-10 INFO Retry
2025-07-10 INFO Success
EOF

# Task 2
cat company_app.log | grep INFO > info.log

# Task 3
cat company_app.log | grep WARN > warn.log

# Task 4
cat company_app.log | grep -E "WARN|ERROR" | wc -l > total_errors.txt
cat total_errors.txt

# Task 5
cat company_app.log | awk '{print $3}' | sort | uniq -c
```

## Lưu ý quan trọng
- Pipe là công cụ mạnh nhất của Linux. DevOps dùng hàng ngày để phân tích log.
- `>` ghi đè file cũ. Luôn kiểm tra kỹ trước khi dùng.
- `awk`, `sed`, `grep`, `sort`, `uniq`, `wc` kết hợp pipe = "vũ khí" xử lý dữ liệu trên terminal.
- `/dev/null` dùng để bỏ qua output/error khi bạn không cần.
- **Storyline:** `company_app.log` của bạn giờ đã có 16 dòng. Đừng xóa nó — sẽ cần ở các bài sau!

---

# Bài 6: Nano & Vim (Soạn Thảo Trong Terminal)

> ⏱️ Thời gian: ~60 phút (15p lý thuyết + 30p thực hành + 15p task)

## Mục tiêu
- Dùng `nano` soạn thảo đơn giản
- Biết cách mở, sửa, lưu, thoát trong `vim`
- Hiểu tại sao DevOps cần biết vim (server thường chỉ có vim)

## Lý thuyết

Trên server Linux, **không có VS Code**. Bạn chỉ có terminal.

| Trình soạn thảo | Ưu điểm | Nhược điểm |
|-----------------|---------|------------|
| **nano** | Dễ dùng, có hướng dẫn ở dưới màn hình | Không mạnh bằng vim |
| **vim** | Mạnh, có ở mọi server, nhiều tính năng | Khó học ban đầu |

> 🎯 DevOps thực tế: SSH vào server → `vim /etc/nginx/nginx.conf` → sửa config → reload. Biết vim là bắt buộc.

## Thực hành từng bước

### Phần A: Nano (15 phút)

#### Bước 1: Mở nano
```bash
cd ~/hoc-linux/devops-roadmap
nano hello.txt
```

Màn hình nano mở ra. Gõ nội dung:
```
Hello from Nano
Day la file thu nghiem
```

#### Bước 2: Lưu và thoát
- **Lưu:** Nhấn `Ctrl + O` (chữ O, không phải số 0), rồi nhấn `Enter`.
- **Thoát:** Nhấn `Ctrl + X`.

#### Bước 3: Kiểm tra file
```bash
cat hello.txt
```

#### Bước 4: Sửa file bằng nano
```bash
nano hello.txt
```

- Dùng phím mũi tên di chuyển con trỏ.
- Thêm dòng mới ở cuối: `DevOps is fun`
- Lưu: `Ctrl + O` → `Enter`
- Thoát: `Ctrl + X`

#### Bước 5: Tìm kiếm trong nano
```bash
nano hello.txt
```
- Nhấn `Ctrl + W` (Where is)
- Gõ: `DevOps`
- Enter → con trỏ nhảy đến chỗ có từ đó.
- Thoát: `Ctrl + X`.

### Phần B: Vim (30 phút)

#### Bước 1: Mở vim
```bash
vim vim_test.txt
```

Màn hình vim mở ra. **Không thể gõ ngay!** Vim có 3 chế độ:

| Chế độ | Cách vào | Cách dùng |
|--------|----------|-----------|
| Normal | Mặc định khi mở vim | Di chuyển, xóa, copy |
| Insert | Nhấn `i` | Gõ văn bản |
| Command | Nhấn `Esc` rồi gõ `:` | Lưu, thoát, tìm kiếm |

#### Bước 2: Vào chế độ Insert và gõ chữ
1. Nhấn `i` (vào Insert mode)
2. Gõ: `Hello from Vim`
3. Nhấn `Esc` (quay về Normal mode)

#### Bước 3: Lưu và thoát
1. Đảm bảo đang ở Normal mode (nhấn `Esc`)
2. Gõ `:wq` → nhấn `Enter`
   - `w` = write (lưu)
   - `q` = quit (thoát)

#### Bước 4: Thoát không lưu (nếu lỡ sửa sai)
1. `Esc`
2. `:q!` → `Enter`
   - `!` = buộc thoát, bỏ thay đổi

#### Bước 5: Mở lại và thêm dòng
```bash
vim vim_test.txt
```

- Nhấn `i` → xuống cuối (phím mũi tên) → gõ thêm: `DevOps roadmap`
- `Esc` → `:wq`

#### Bước 6: Xóa dòng trong vim
```bash
vim vim_test.txt
```

- Ở Normal mode (nhấn `Esc`)
- Di chuyển đến dòng muốn xóa (phím mũi tên)
- Nhấn `dd` → dòng biến mất
- `:wq` để lưu

#### Bước 7: Tìm kiếm trong vim
```bash
vim vim_test.txt
```

- Normal mode
- Gõ `/DevOps` → nhấn `Enter`
- Vim highlight từ `DevOps`
- Nhấn `n` để tìm tiếp theo
- Nhấn `N` (shift+n) để tìm ngược lại

#### Bước 8: Copy & paste trong vim
```bash
vim vim_test.txt
```

- Di chuyển đến dòng muốn copy
- `yy` (yank line) = copy dòng
- Di chuyển đến chỗ muốn paste
- `p` = paste sau dòng hiện tại

> 💡 Vim còn rất nhiều tính năng nâng cao. Bạn chỉ cần nhớ: `i` để gõ, `Esc` để về Normal, `:wq` để lưu, `:q!` để thoát, `/` để tìm, `dd` để xóa dòng.

#### Bước 9: Sửa `deploy.sh` bằng vim — Storyline

```bash
cd ~/hoc-linux/devops-roadmap
vim deploy.sh
```

- Normal mode
- Di chuyển xuống cuối file (`G` — shift+g)
- `o` = mở dòng mới bên dưới và vào Insert mode
- Gõ: `echo "Deploy hoan tat luc $(date)"`
- `Esc`
- `:wq`

Kiểm tra:
```bash
cat deploy.sh
./deploy.sh
```

Output mới:
```
Dang deploy...
Moi truong: development
Deploy hoan tat luc Thu Jul 16 ...
```

#### Bước 10: Sửa `company_app.log` bằng vim — Storyline

```bash
vim company_app.log
```

- Normal mode
- Tìm dòng có chữ `timeout`: `/timeout` → `Enter`
- Sửa thành `TIMEOUT` (chữ hoa): `i` → sửa → `Esc`
- `:wq`

Kiểm tra:
```bash
grep -i timeout company_app.log
```

> 🎯 **Storyline:** Giờ bạn đã biết cách sửa file production bằng vim. `deploy.sh` và `company_app.log` sẽ tiếp tục được dùng ở Giai đoạn 2 và 3.

## Task cuối bài (15 phút)

1. Dùng `nano` tạo file `nano_bai_tap.txt`, gõ tên bạn và ngày hôm nay, lưu lại.
2. Dùng `vim` mở file đó, xóa dòng ngày, thêm dòng `Hoc Linux cho DevOps`, lưu.
3. Dùng `vim` tạo file `vim_bai_tap.txt`, gõ 3 dòng:
   ```
   Dong 1
   Dong 2
   Dong 3
   ```
   Copy dòng 1 và paste xuống cuối, sau đó lưu.
4. **(Storyline)** Kiểm tra lại `deploy.sh` và `company_app.log` bằng `cat` để chắc đã sửa đúng.
5. Chạy thử `./deploy.sh` với `ENV=staging`.

## Đáp án / Gợi ý

```bash
cd ~/hoc-linux/devops-roadmap

# Task 1
nano nano_bai_tap.txt
# Gõ: Hoang
# Gõ: 16/07/2026
# Ctrl+O → Enter → Ctrl+X

# Task 2
vim nano_bai_tap.txt
# Di chuyển đến dòng ngày
# dd (xóa dòng)
# i → gõ: Hoc Linux cho DevOps
# Esc → :wq

# Task 3
vim vim_bai_tap.txt
# i → gõ 3 dòng
# Esc → di chuyển lên dòng 1
# yy → di chuyển xuống cuối → p
# :wq

# Task 4
cat nano_bai_tap.txt
cat vim_bai_tap.txt
cat deploy.sh
cat company_app.log

# Task 5
ENV=staging ./deploy.sh
```

## Lưu ý quan trọng
- **Nano** phù hợp khi bạn mới bắt đầu hoặc cần sửa nhanh file config đơn giản.
- **Vim** có ở mọi server Linux. Khi SSH vào server production, `vim` là lựa chọn duy nhất.
- DevOps hay sửa file: `/etc/hosts`, `/etc/nginx/nginx.conf`, `/etc/docker/daemon.json`, `~/.ssh/config` — tất cả đều bằng vim.
- Nếu vim hiện màu xanh lạ, đó là chế độ "visual". Nhấn `Esc` nhiều lần rồi `:q!` để thoát.

---

## ✅ Kết Thúc Giai Đoạn 1

Sau 6 bài, bạn đã biết:
- Quản lý quyền file (`chmod`, `chown`)
- Quản lý tiến trình (`ps`, `top`, `kill`)
- Cài phần mềm (`apt`)
- Biến môi trường & alias (`export`, `alias`, `.bashrc`)
- Pipe & Redirect (`|`, `>`, `>>`, `2>`)
- Soạn thảo (`nano`, `vim`)

**Hãy ôn lại bằng cách:**
```bash
cd ~/hoc-linux/devops-roadmap
alias
cat company_app.log | grep ERROR | wc -l
ll
./deploy.sh
```

Sẵn sàng? Mở file `02-giai-doan-2-linux-cho-devops.md` để học tiếp! 🚀
