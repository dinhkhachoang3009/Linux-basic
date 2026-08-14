# Linux DevOps Roadmap - Hướng Dẫn Sử Dụng

> Dành cho: Hoang | Môi trường: Ubuntu 26.04 LTS (dual boot) | Mục tiêu: DevOps

---

## 📁 Cấu Trúc Thư Mục

```
linux_devops_roadmap/
├── 00-huong-dan-su-dung.md              ← Bạn đang đọc file này
├── linux_ubuntu_hoc_tap.md              ← Tổng kết & tiến độ học tập
├── 01-giai-doan-1-linux-core.md         ← 7 bài căn bản (có bài 0 làm quen lệnh)
├── 02-giai-doan-2-linux-cho-devops.md   ← 8 bài chuyên sâu
├── 03-giai-doan-3-thuc-chien-devops.md  ← 4 bài thực chiến
└── 99-cheat-sheet.md                    ← Bảng tra nhanh
```

---

## 🖥️ Chuẩn Bị Trước Khi Học

### 1. Kiểm tra bạn đang ở đâu
Mở terminal trong Ubuntu và chạy:
```bash
whoami
pwd
```

Kết quả mong đợi:
```
hoang
/home/hoang
```

### 2. Tạo thư mục học tập riêng (chỉ cần làm 1 lần)
```bash
mkdir -p ~/Test
cd ~/Test
pwd
```

> 💡 **Mẹo:** Từ giờ, mỗi lần học bạn chỉ cần mở terminal và chạy:
> ```bash
> cd ~/Test
> ```

---

## 📖 Cách Học Với Roadmap Này

Mỗi bài học được thiết kế theo công thức **~60 phút**:

| Giai đoạn | Thời gian | Bạn làm gì |
|-----------|-----------|------------|
| Đọc lý thuyết | ~15 phút | Đọc phần giải thích, ghi nhớ khái niệm |
| Thực hành từng bước | ~30 phút | Mở terminal, gõ **từng lệnh** theo hướng dẫn, quan sát output |
| Task cuối bài | ~15 phút | Tự làm bài tập nhỏ, kiểm tra đáp án |

### Quy tắc vàng khi học:
1. **Đừng copy paste cả loạt lệnh** — gõ từng lệnh một để nhớ.
2. **Nếu lệnh báo lỗi** — đọc kỹ lỗi, kiểm tra lại bước trước đó.
3. **Nếu không hiểu output** — chạy lại lệnh kèm `man <lệnh>` để đọc hướng dẫn (ví dụ: `man ls`).
4. **Luôn kiểm tra vị trí hiện tại** trước khi xóa/sửa: `pwd`.
5. **An toàn với `rm -rf`** — câu lệnh này xóa vĩnh viễn, không có thùng rác.

---

## 🎬 Dòng Chảy Thực Hành Liên Tục (Storyline)

Roadmap này không chỉ là lý thuyết rời rạc — bạn sẽ **nuôi dưỡng 2 file xuyên suốt** như đang quản lý một dự án thật:

| File | Tạo ở | Tái sử dụng ở |
|------|-------|--------------|
| **`deploy.sh`** | Bài 1 (chmod +x) | Bài 4 (thêm biến môi trường), Bài 6 (sửa bằng vim), Bài 15 (nâng cấp script thực tế), Bài 17 (đưa vào pipeline) |
| **`company_app.log`** | Bài 5 (tạo log mẫu) | Bài 6 (sửa typo bằng vim), Bài 13 (tail -f + logrotate), Bài 14 (chown + find), Bài 15 (script phân tích log), Bài 18 (audit) |

> 💡 **Ý nghĩa:** Bạn không chỉ học lệnh — bạn thấy cách một file "sống" qua nhiều ngày, nhiều thao tác, giống hệt môi trường production.

---

## 🚀 Lộ Trình 3 Giai Đoạn

### Giai đoạn 1: Linux Core (Tuần 1-2)
Làm quen sâu với Linux, từ những lệnh cơ bản nhất đến các kỹ năng cần thiết.

- **Bài 0: Làm quen lệnh cơ bản** — `ls`, `cd`, `pwd`, `mkdir`, `touch`, `rm`, `cp`, `mv`, `cat` — làm hoàn toàn trong `~/Test`
- Bài 1: Permissions (quyền truy cập file) — **tạo `deploy.sh`**
- Bài 2: Process Management (quản lý tiến trình)
- Bài 3: Package Management (cài đặt phần mềm)
- Bài 4: Environment & Alias (biến môi trường) — **thêm biến vào `deploy.sh`**
- Bài 5: Pipe & Redirect (công cụ cốt lõi của DevOps) — **tạo `company_app.log`**
- Bài 6: Nano & Vim (soạn thảo trong terminal) — **sửa `deploy.sh` và `company_app.log`**

### Giai đoạn 2: Linux cho DevOps (Tuần 3-5)
Học những gì DevOps engineer dùng hàng ngày trên server.

- Bài 7: User & Group + Sudo
- Bài 8: Systemd & Service
- Bài 9: Disk & Storage
- Bài 10: Networking sâu hơn
- Bài 11: SSH & Remote
- Bài 12: Cron & Automation
- Bài 13: Log Management — **theo dõi `company_app.log` + logrotate**
- Bài 14: File Ownership & Advanced Find — **tìm & đổi quyền `company_app.log`**

### Giai đoạn 3: Thực chiến DevOps trên Ubuntu (Tuần 6-7)
Kết hợp tất cả để viết script, làm việc với container, giả lập CI/CD.

- Bài 15: Shell Script thực tế — **viết script phân tích `company_app.log`, nâng cấp `deploy.sh`**
- Bài 16: Docker trên Ubuntu — *Yêu cầu cài Docker Engine trước*
- Bài 17: Tự động hóa deploy giả lập — **đưa `deploy.sh` vào pipeline**
- Bài 18: Tổng hợp & Ôn tập phỏng vấn — **audit script đọc `company_app.log`**

---

## ⚠️ Lưu Ý An Toàn

- Bạn đang học trên Ubuntu dual boot thật — **mọi thay đổi đều có tác động thực sự**. Hãy cẩn thận với lệnh xóa/sửa hệ thống.
- Thoải mái thực hành trong `/home/hoang/Test` — đây là không gian an toàn của bạn.
- **Không** chạy `rm -rf /`, `sudo chown -R user /`, hoặc các lệnh "nguy hiểm" trừ khi bạn thực sự hiểu hậu quả.
- Nếu lỡ làm hỏng hệ thống, bạn có thể cần cài lại Ubuntu — nhưng dữ liệu trong `/home/hoang` thường an toàn nếu bạn có phân vùng home riêng.

---

## ✅ Checklist Trước Khi Bắt Đầu Bài 0

```bash
# Chạy từng lệnh này trong terminal để kiểm tra:
whoami          # Kết quả: hoang
pwd             # Kết quả: /home/hoang
cd ~            # Về home
mkdir -p ~/Test
cd ~/Test
ls -la          # Thư mục trống hoặc có vài file
```

Nếu tất cả lệnh trên chạy ok → **Sẵn sàng!** Mở file `01-giai-doan-1-linux-core.md` và bắt đầu Bài 0.

Chúc bạn học tập hiệu quả! 🎯
