# 📚 Tổng Kết Học Linux WSL - Hoang

> Cập nhật: 03/07/2026 | WSL Ubuntu 26.04 LTS | User: `hoang`

---

## 🖥️ Môi Trường Hiện Tại

```
Windows 11
├── Docker Desktop        ← độc lập, không bị ảnh hưởng bởi WSL
├── Git (Windows)         ← độc lập
├── Python (Windows)      ← độc lập
└── WSL2 - Ubuntu 26.04  ← môi trường học Linux
    └── User: hoang
        └── Home: /home/hoang
```

### Cấu trúc file đang có trong WSL
```
/home/hoang/
├── bai2/
│   ├── hello.txt          ← file thực hành (có nội dung + ERROR/INFO logs)
│   └── backup/
│       ├── hello.txt
│       └── hello_v2.txt
└── thu_muc_test/          ← có thể xóa: rm -rf ~/thu_muc_test
    ├── file1.txt
    ├── file2.txt
    └── file3.txt
```

---

## ✅ Đã Học - 4 Bài

### Bài 1: Navigation
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

### Bài 2: Đọc & Ghi File
| Lệnh | Ý nghĩa |
|------|---------|
| `echo "text" > file` | Ghi vào file (ghi đè) ⚠️ |
| `echo "text" >> file` | Ghi thêm vào file (an toàn) |
| `cat file` | Đọc toàn bộ file |
| `head -n file` | Đọc n dòng đầu |
| `tail -n file` | Đọc n dòng cuối |

### Bài 3: Copy, Move, Rename
| Lệnh | Ý nghĩa |
|------|---------|
| `cp file1 file2` | Copy file |
| `cp file thư_mục/` | Copy vào thư mục |
| `mv tên_cũ tên_mới` | Đổi tên file |
| `mv file thư_mục/` | Di chuyển file |

### Bài 4: Tìm Kiếm
| Lệnh | Ý nghĩa |
|------|---------|
| `grep "từ" file` | Tìm dòng chứa từ khóa |
| `grep "từ" file -c` | Đếm số dòng khớp |
| `grep -i "từ" file` | Tìm không phân biệt hoa/thường |
| `grep -v "từ" file` | Tìm dòng KHÔNG chứa từ khóa |
| `find ~ -name "file"` | Tìm file theo tên |
| `find ~ -name "*.txt"` | Tìm theo pattern |
| `find ~ -type d -name "tên"` | Tìm thư mục |

---

## 📅 Kế Hoạch Học Tiếp

### Bài 5: Permissions (tiếp theo)
```bash
# Đọc permission
ls -la hello.txt
# -rw-r--r--  =  owner(rw-) group(r--) others(r--)

# Đổi permission
chmod 777 file    # tất cả đọc/ghi/chạy
chmod 644 file    # bình thường cho file
chmod +x file     # thêm quyền execute (chạy script)

# Chạy script bash
echo '#!/bin/bash' > test.sh
echo 'echo "Hello!"' >> test.sh
chmod +x test.sh
./test.sh
```

### Bài 6: Process Management
```bash
ps aux            # xem tất cả process đang chạy
top / htop        # monitor realtime
kill PID          # dừng process theo ID
kill -9 PID       # buộc dừng
jobs              # xem job đang chạy
Ctrl+C            # dừng process đang chạy
Ctrl+Z            # tạm dừng process
```

### Bài 7: Networking cơ bản
```bash
ping google.com           # kiểm tra kết nối
curl https://example.com  # gọi HTTP request
wget url                  # tải file từ URL
netstat -tulpn            # xem port đang mở
ss -tulpn                 # thay thế netstat (mới hơn)
```

### Bài 8: Shell Scripting cơ bản
```bash
#!/bin/bash
# Biến
TEN="Hoang"
echo "Xin chao $TEN"

# Vòng lặp
for i in 1 2 3; do
  echo "So: $i"
done

# Điều kiện
if [ -f "file.txt" ]; then
  echo "File tồn tại"
fi
```

### Bài 9: Pipe & Redirect (rất quan trọng với DevOps)
```bash
# Pipe - nối output của lệnh này vào lệnh khác
cat file.txt | grep "ERROR"
cat file.txt | grep "ERROR" | wc -l   # đếm dòng lỗi

# Redirect
command > output.txt    # ghi output ra file
command 2> error.txt    # ghi error ra file
command &> all.txt      # ghi cả output lẫn error
```

### Bài 10: Vim / Nano - Trình soạn thảo trong terminal
```bash
nano file.txt     # dễ dùng hơn, phù hợp người mới
vim file.txt      # mạnh hơn, cần học thêm

# Nano: Ctrl+O lưu, Ctrl+X thoát
# Vim:  i để insert, Esc để thoát insert, :wq để lưu và thoát
```

---

## ⚠️ Lưu Ý Quan Trọng

- `rm -rf` không có thùng rác, **xóa vĩnh viễn**
- `>` ghi đè file cũ, dùng `>>` để an toàn hơn
- Luôn `pwd` trước khi chạy lệnh xóa/sửa
- WSL và Windows **tách biệt hoàn toàn** - an toàn để thực hành
- Chỉ đụng đến project Docker khi `cd /mnt/c/...`

---

## 🚀 Tiếp Tục Từ Đây

> **UPDATE:** Đã có roadmap DevOps chi tiết tại `C:\Users\pc\HUST\linux_devops_roadmap\`
> 
> Gồm 18 bài, mỗi bài ~60 phút, có lý thuyết + thực hành từng bước + task cuối bài.
> 
> Hãy mở file `00-huong-dan-su-dung.md` để bắt đầu!

```bash
# Mở WSL, vào thư mục thực hành
cd ~/bai2

# Dọn dẹp thư mục thừa
rm -rf ~/thu_muc_test

# Bắt đầu Bài 5: Permissions
ls -la hello.txt
```
