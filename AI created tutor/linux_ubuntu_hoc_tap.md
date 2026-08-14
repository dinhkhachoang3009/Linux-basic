# 📚 Tổng Kết Học Linux Ubuntu - Hoang

> Cập nhật: 10/08/2026 | Ubuntu 26.04 LTS (dual boot) | User: `hoang`

---

## 🖥️ Môi Trường Hiện Tại

```
Ubuntu 26.04 (dual boot)
├── Terminal (GNOME/Konsole/...) ← môi trường học Linux
└── User: hoang
    └── Home: /home/hoang
    └── Thư mục thực hành: /home/hoang/Test
```

### Cấu trúc file đang có trong thư mục thực hành
```
~/Test/
├── deploy.sh              ← script storyline (tạo ở Bài 1)
├── company_app.log        ← file log storyline (tạo ở Bài 5)
└── ... (các file bài tập khác)
```

---

## ✅ Đã Học - Các Bài Hoàn Thành

### Bài 0: Lệnh Cơ Bản
| Lệnh | Ý nghĩa |
|------|---------|
| `pwd` | Xem đang ở thư mục nào |
| `ls` | Liệt kê file |
| `ls -l` | Liệt kê chi tiết |
| `ls -la` | Liệt kê cả file ẩn |
| `cd /` | Đi đến thư mục gốc |
| `cd ~` | Về home (`/home/hoang`) |
| `cd ..` | Lùi 1 cấp |
| `mkdir tên` | Tạo thư mục |
| `mkdir -p a/b/c` | Tạo nhiều cấp |
| `touch file.txt` | Tạo file rỗng |
| `rmdir tên` | Xóa thư mục rỗng |
| `rm -rf tên` | Xóa thư mục có nội dung ⚠️ |
| `cp file1 file2` | Copy file |
| `mv tên_cũ tên_mới` | Đổi tên / di chuyển |
| `cat file` | Đọc toàn bộ file |
| `head -n file` | Đọc n dòng đầu |
| `tail -n file` | Đọc n dòng cuối |
| `echo "text" > file` | Ghi vào file (ghi đè) ⚠️ |
| `echo "text" >> file` | Ghi thêm vào file (an toàn) |
| `grep "từ" file` | Tìm dòng chứa từ khóa |
| `find ~ -name "file"` | Tìm file theo tên |

### Bài 1: Permissions
| Lệnh | Ý nghĩa |
|------|---------|
| `chmod 755 file` | Owner: rwx, Group/Others: rx |
| `chmod 644 file` | Bình thường cho file |
| `chmod +x file` | Thêm quyền execute |
| `ls -la` | Xem quyền chi tiết |

### Các bài tiếp theo... (cập nhật khi hoàn thành)

---

## 📅 Kế Hoạch Học Tiếp

### Bài 2: Process Management
```bash
ps aux            # xem tất cả process đang chạy
top / htop        # monitor realtime
kill PID          # dừng process theo ID
kill -9 PID       # buộc dừng
jobs              # xem job đang chạy
Ctrl+C            # dừng process đang chạy
Ctrl+Z            # tạm dừng process
```

### Bài 3: Package Management
```bash
sudo apt update   # cập nhật danh sách package
sudo apt install  # cài đặt
sudo apt remove   # gỡ bỏ
```

### Bài 4: Environment & Alias
```bash
export VAR=value  # tạo biến môi trường
echo $VAR         # đọc biến
alias ll='ls -lah' # tạo alias
source ~/.bashrc  # áp dụng thay đổi
```

### Bài 5: Pipe & Redirect
```bash
cat file | grep "ERROR"       # pipe cơ bản
command > output.txt          # redirect output
command 2> error.txt          # redirect lỗi
command &> all.txt            # redirect cả 2
cat file | grep ERROR | wc -l # pipe nhiều tầng
```

### Bài 6: Vim / Nano
```bash
nano file.txt     # dễ dùng, phù hợp người mới
vim file.txt      # mạnh hơn, cần học thêm
```

---

## ⚠️ Lưu Ý Quan Trọng

- `rm -rf` không có thùng rác, **xóa vĩnh viễn**
- `>` ghi đè file cũ, dùng `>>` để an toàn hơn
- Luôn `pwd` trước khi chạy lệnh xóa/sửa
- Thực hành hoàn toàn trong `~/Test` để tránh ảnh hưởng hệ thống
- Các lệnh có `sudo` ảnh hưởng hệ thống — hãy chắc chắn trước khi chạy

---

## 🚀 Tiếp Tục Từ Đây

> Hãy mở file `01-giai-doan-1-linux-core.md` để bắt đầu từ Bài 0!

```bash
# Mở terminal, vào thư mục thực hành
cd ~/Test

# Bắt đầu Bài 0: Lệnh cơ bản
pwd
ls -la
```
