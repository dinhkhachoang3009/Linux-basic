# Cheat Sheet — Linux DevOps

> Tra cứu nhanh khi cần nhớ lại lệnh

---

## 📁 File & Thư Mục

| Lệnh | Ý nghĩa |
|------|---------|
| `pwd` | Xem đường dẫn hiện tại |
| `ls -lah` | Liệt kê file chi tiết |
| `cd ~` | Về home |
| `cd -` | Về thư mục trước đó |
| `mkdir -p a/b/c` | Tạo thư mục nhiều cấp |
| `touch file.txt` | Tạo file rỗng |
| `cp -r src/ dest/` | Copy thư mục đệ quy |
| `mv old new` | Đổi tên / di chuyển |
| `rm -rf thu_muc` | Xóa thư mục (⚠️ cẩn thận) |
| `cat file` | Đọc file |
| `head -n 20 file` | 20 dòng đầu |
| `tail -n 20 file` | 20 dòng cuối |
| `tail -f file` | Theo dõi file realtime |
| `echo "text"` | In ra màn hình |
| `echo "text" > file` | Ghi đè file |
| `echo "text" >> file` | Ghi thêm vào file |
| `less file` | Xem file có thể cuộn (nhấn `q` thoát) |

---

## 🔒 Permissions

| Lệnh | Ý nghĩa |
|------|---------|
| `chmod 755 file` | Owner: rwx, Group/Others: rx |
| `chmod 644 file` | Owner: rw, Group/Others: r |
| `chmod +x file` | Thêm quyền chạy |
| `chown user:group file` | Đổi owner + group |
| `chown -R user dir/` | Đổi owner đệ quy |
| `ls -la` | Xem quyền chi tiết |

**Bảng số:**
- `7` = rwx | `6` = rw- | `5` = r-x | `4` = r-- | `0` = ---

---

## 🔍 Tìm Kiếm & Lọc

| Lệnh | Ý nghĩa |
|------|---------|
| `grep "word" file` | Tìm dòng chứa từ |
| `grep -i "word" file` | Không phân biệt hoa thường |
| `grep -v "word" file` | Dòng KHÔNG chứa từ |
| `grep -c "word" file` | Đếm số dòng khớp |
| `find . -name "*.txt"` | Tìm file theo tên |
| `find . -type d -name "logs"` | Tìm thư mục |
| `find . -size +10M` | Tìm file > 10MB |
| `find . -mtime +7` | File sửa > 7 ngày trước |
| `find . -exec rm {} \;` | Chạy lệnh trên kết quả |

---

## 🔗 Pipe & Redirect

| Ký hiệu | Ý nghĩa |
|---------|---------|
| `\|` | Nối output → input lệnh sau |
| `>` | Ghi đè file |
| `>>` | Ghi thêm vào file |
| `2>` | Ghi lỗi ra file |
| `&>` | Ghi cả output + lỗi |
| `<` | Lấy file làm input |
| `2>&1` | Chuyển lỗi thành output (để ghi chung) |
| `/dev/null` | "Hố đen" — bỏ output |

**Ví dụ DevOps:**
```bash
cat company_app.log | grep ERROR | wc -l
command > out.txt 2> err.txt
command &> all.txt
command > /dev/null 2>&1
```

---

## ⚙️ Process Management

| Lệnh | Ý nghĩa |
|------|---------|
| `ps aux` | Xem tất cả process |
| `top` | Giám sát realtime (q để thoát) |
| `htop` | Đẹp hơn top |
| `kill PID` | Dừng process |
| `kill -9 PID` | Buộc dừng |
| `pkill tên` | Dừng theo tên |
| `pgrep tên` | Tìm PID |
| `jobs` | Xem job nền |
| `bg %1` | Chạy lại job nền |
| `fg %1` | Đưa job lên foreground |
| `Ctrl+C` | Ngắt process |
| `Ctrl+Z` | Tạm dừng process |

---

## 📦 Package & Disk

| Lệnh | Ý nghĩa |
|------|---------|
| `sudo apt update` | Cập nhật danh sách package |
| `sudo apt install -y pkg` | Cài package |
| `sudo apt remove pkg` | Gỡ package |
| `apt search từ` | Tìm package |
| `df -h` | Xem dung lượng ổ đĩa |
| `du -sh dir/` | Xem dung lượng thư mục |
| `du -h --max-depth=1` | Chi tiết từng thư mục con |

---

## 👤 User & Sudo

| Lệnh | Ý nghĩa |
|------|---------|
| `whoami` | Tên user hiện tại |
| `id` | Thông tin user/group |
| `sudo <lệnh>` | Chạy với quyền admin |
| `sudo su -` | Chuyển sang root |
| `useradd -m -s /bin/bash user` | Tạo user |
| `passwd user` | Đặt mật khẩu |
| `usermod -aG sudo user` | Thêm user vào nhóm sudo |

---

## 🖥️ Systemd & Service

| Lệnh | Ý nghĩa |
|------|---------|
| `systemctl status svc` | Xem trạng thái |
| `systemctl start svc` | Bật |
| `systemctl stop svc` | Tắt |
| `systemctl restart svc` | Khởi động lại |
| `systemctl enable svc` | Tự chạy khi boot |
| `systemctl disable svc` | Không tự chạy |
| `journalctl -u svc` | Xem log service |
| `journalctl -u svc -f` | Theo dõi log realtime |
| `journalctl --since today` | Log hôm nay |

---

## 🌐 Network

| Lệnh | Ý nghĩa |
|------|---------|
| `ip a` | Xem IP |
| `hostname -I` | Xem IP nhanh |
| `ping -c 4 host` | Test kết nối |
| `curl -I url` | Xem header HTTP |
| `wget url -O file` | Tải file |
| `ss -tulpn` | Xem port đang mở |
| `netstat -tulpn` | Tương tự ss |
| `nslookup host` | Tra DNS |

---

## ⏰ Cron

| Lệnh | Ý nghĩa |
|------|---------|
| `crontab -l` | Xem lịch hiện tại |
| `crontab -e` | Sửa lịch |
| `* * * * * cmd` | Mỗi phút |
| `0 2 * * * cmd` | 2h sáng mỗi ngày |
| `*/5 * * * * cmd` | Mỗi 5 phút |
| `@reboot cmd` | Khi khởi động |

---

## 🔑 SSH & Remote

| Lệnh | Ý nghĩa |
|------|---------|
| `ssh-keygen -t rsa` | Tạo SSH key |
| `ssh user@server` | Đăng nhập |
| `scp file user@server:/path` | Copy file lên server |
| `rsync -avz src/ dest/` | Đồng bộ mạnh mẽ |
| `ssh-copy-id server` | Copy public key |

---

## 📝 Vim & Nano

### Nano
- `Ctrl+O` → Lưu
- `Ctrl+X` → Thoát
- `Ctrl+W` → Tìm kiếm

### Vim
- `i` → Insert mode (gõ chữ)
- `Esc` → Normal mode
- `:wq` → Lưu và thoát
- `:q!` → Thoát không lưu
- `dd` → Xóa dòng
- `yy` → Copy dòng
- `p` → Paste
- `/từ` → Tìm kiếm
- `n` → Tìm tiếp theo

---

## 🐳 Docker Cơ Bản

| Lệnh | Ý nghĩa |
|------|---------|
| `docker ps` | Container đang chạy |
| `docker ps -a` | Tất cả container |
| `docker run -d -p 80:80 nginx` | Chạy container nền |
| `docker logs -f container` | Xem log realtime |
| `docker exec -it container bash` | Vào trong container |
| `docker stop container` | Dừng |
| `docker rm container` | Xóa |
| `docker images` | Xem image |
| `docker rmi image` | Xóa image |
| `-v host:container` | Mount volume |

---

## 💡 Mẹo DevOps

1. **Luôn kiểm tra trước khi xóa:**
   ```bash
   ls target*        # Kiểm tra trước
   rm target*        # Rồi mới xóa
   ```

2. **Script an toàn:**
   ```bash
   #!/bin/bash
   set -e            # Dừng khi lỗi
   set -u            # Báo lỗi nếu biến chưa định nghĩa
   ```

3. **Tạo backup trước khi sửa config:**
   ```bash
   cp config.conf config.conf.bak.$(date +%s)
   ```

4. **Kiểm tra syntax script:**
   ```bash
   bash -n script.sh
   ```

5. **Tìm lỗi nhanh:**
   ```bash
   journalctl -u myapp --since "1 hour ago" | grep ERROR
   ```

---

> 📌 Lưu file này lại. Mỗi khi quên lệnh → mở ra tra!
