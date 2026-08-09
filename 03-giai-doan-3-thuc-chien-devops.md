# Giai Đoạn 3: Thực Chiến DevOps Trên Ubuntu

> 4 bài học | Mỗi bài ~60 phút | Mục tiêu: Kết hợp tất cả kỹ năng để tự động hóa như DevOps thực thụ

---

# Bài 15: Shell Script Thực Tế (Backup + Health Check)

> ⏱️ Thời gian: ~60 phút (15p lý thuyết + 30p thực hành + 15p task)

## Mục tiêu
- Viết script backup tự động
- Viết script health check hệ thống
- Dùng biến, điều kiện, vòng lặp trong bash
- Đặt quyền chạy và test script

## Lý thuyết

Script bash là file chứa nhiều lệnh Linux. DevOps viết script để:
- Backup dữ liệu định kỳ
- Kiểm tra server health (CPU, RAM, disk)
- Deploy ứng dụng
- Dọn dẹp log, file tạm

Cấu trúc script cơ bản:
```bash
#!/bin/bash
# Comment
echo "Hello"
```

> 🎯 DevOps thực tế: Mọi pipeline CI/CD cuối cùng đều chạy shell script trên server.

## Thực hành từng bước

### Bước 1: Script backup đơn giản
```bash
cd ~/Test

cat > backup_daily.sh << 'EOF'
#!/bin/bash

# Thiet lap
SOURCE_DIR="/home/hoang/Test"
BACKUP_DIR="/home/hoang/Test/backups"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_NAME="backup_${DATE}.tar.gz"

# Tao thu muc backup neu chua co
mkdir -p "$BACKUP_DIR"

# Nen va luu backup
tar -czf "${BACKUP_DIR}/${BACKUP_NAME}" "$SOURCE_DIR" 2>/dev/null

# Ghi log
echo "[$(date)] Backup created: ${BACKUP_NAME}" >> "${BACKUP_DIR}/backup.log"
echo "Backup done: ${BACKUP_NAME}"
EOF
```

Cho phép chạy:
```bash
chmod +x backup_daily.sh
```

Chạy thử:
```bash
./backup_daily.sh
```

Kiểm tra:
```bash
ls -lh backups/
cat backups/backup.log
```

### Bước 2: Script health check hệ thống
```bash
cat > health_check.sh << 'EOF'
#!/bin/bash

LOG_FILE="/home/hoang/Test/health_check.log"

# Lay thong tin
CPU=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d'%' -f1)
MEM=$(free | grep Mem | awk '{print $3/$2 * 100.0}')
DISK=$(df -h / | tail -1 | awk '{print $5}' | cut -d'%' -f1)

# Ghi log
echo "[$(date)] CPU: ${CPU}% | MEM: ${MEM}% | DISK: ${DISK}%" >> "$LOG_FILE"

# Kiem tra canh bao
if [ "$DISK" -gt 80 ]; then
    echo "[$(date)] WARNING: Disk usage > 80%" >> "$LOG_FILE"
fi

if [ "${MEM%.*}" -gt 90 ]; then
    echo "[$(date)] WARNING: Memory usage > 90%" >> "$LOG_FILE"
fi

echo "Health check done. See ${LOG_FILE}"
EOF
```

> Giải thích:
> - `top -bn1` = chạy 1 lần, batch mode
> - `awk '{print $2}'` = lấy cột 2
> - `cut -d'%' -f1` = cắt lấy phần trước dấu %
> - `free | grep Mem` = lấy dòng memory
> - `[ "$DISK" -gt 80 ]` = kiểm tra điều kiện số nguyên

Cho phép chạy và test:
```bash
chmod +x health_check.sh
./health_check.sh
cat health_check.log
```

### Bước 3: Script với tham số đầu vào
```bash
cat > greet.sh << 'EOF'
#!/bin/bash

if [ -z "$1" ]; then
    echo "Usage: ./greet.sh <ten>"
    exit 1
fi

echo "Xin chao, $1!"
echo "Hom nay la: $(date)"
EOF

chmod +x greet.sh
./greet.sh Hoang
./greet.sh
```

Output lần 2:
```
Usage: ./greet.sh <ten>
```

### Bước 4: Vòng lặp trong bash
```bash
cat > loop_demo.sh << 'EOF'
#!/bin/bash

echo "=== Danh sach file .txt ==="
for file in *.txt; do
    if [ -f "$file" ]; then
        echo "File: $file - Size: $(wc -l < $file) lines"
    fi
done
EOF

chmod +x loop_demo.sh
./loop_demo.sh
```

### Bước 5: Hàm (function) trong bash
```bash
cat > functions_demo.sh << 'EOF'
#!/bin/bash

log_message() {
    local level=$1
    local message=$2
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] [$level] $message"
}

log_message "INFO" "Bat dau script"
log_message "WARN" "Disk gan day"
log_message "ERROR" "Khong ket noi DB"
EOF

chmod +x functions_demo.sh
./functions_demo.sh
```

### Bước 6: Script phân tích `company_app.log` — Storyline

```bash
cat > analyze_log.sh << 'EOF'
#!/bin/bash

LOG_FILE="/home/hoang/Test/company_app.log"
REPORT_FILE="/home/hoang/Test/log_report.txt"

if [ ! -f "$LOG_FILE" ]; then
    echo "ERROR: $LOG_FILE khong ton tai"
    exit 1
fi

TOTAL=$(wc -l < "$LOG_FILE")
ERRORS=$(grep -c "ERROR" "$LOG_FILE")
WARNS=$(grep -c "WARN" "$LOG_FILE")
INFOS=$(grep -c "INFO" "$LOG_FILE")

{
    echo "=== LOG ANALYSIS REPORT ==="
    echo "File: $LOG_FILE"
    echo "Time: $(date)"
    echo "Total lines: $TOTAL"
    echo "ERROR: $ERRORS"
    echo "WARN: $WARNS"
    echo "INFO: $INFOS"
    echo "==========================="
} > "$REPORT_FILE"

cat "$REPORT_FILE"
EOF

chmod +x analyze_log.sh
./analyze_log.sh
```

> 📌 **Storyline:** Script này đọc file log bạn đã "nuôi" từ Bài 5, 6, 13, 14 và tạo báo cáo. Đây là công việc DevOps làm hàng ngày — phân tích log để tìm vấn đề.

### Bước 7: Nâng cấp `deploy.sh` — Storyline

```bash
cat > deploy.sh << 'EOF'
#!/bin/bash

set -e

ENV="${ENV:-development}"
LOG_FILE="/home/hoang/Test/deploy.log"

deploy() {
    echo "[$(date)] Bat dau deploy moi truong: $ENV" | tee -a "$LOG_FILE"
    echo "[$(date)] Deploy thanh cong" | tee -a "$LOG_FILE"
}

if [ "$ENV" = "production" ]; then
    echo "WARNING: Deploy production!"
fi

deploy
EOF

chmod +x deploy.sh
ENV=staging ./deploy.sh
cat deploy.log
```

> 🎯 **Storyline:** `deploy.sh` của bạn giờ đã là một script DevOps thực sự — có function, log, kiểm tra môi trường. Bạn sẽ đưa nó vào pipeline ở Bài 17.

## Task cuối bài (15 phút)

1. Viết script `disk_alert.sh` kiểm tra disk usage của `/`. Nếu > 50% thì in ra cảnh báo, ngược lại in "Disk OK".
2. Viết script `count_files.sh` đếm số file `.txt`, `.sh`, `.log` trong thư mục hiện tại (dùng vòng lặp).
3. **(Storyline)** Chạy lại `analyze_log.sh` và đối chiếu số lượng ERROR với `company_app.log` bằng `grep`.
4. **(Storyline)** Chạy `./deploy.sh` với `ENV=production` và xem output.

## Đáp án / Gợi ý

```bash
cd ~/Test

# Task 1
cat > disk_alert.sh << 'EOF'
#!/bin/bash
USAGE=$(df -h / | tail -1 | awk '{print $5}' | cut -d'%' -f1)
if [ "$USAGE" -gt 50 ]; then
    echo "WARNING: Disk usage is ${USAGE}%"
else
    echo "Disk OK: ${USAGE}%"
fi
EOF
chmod +x disk_alert.sh
./disk_alert.sh

# Task 2
cat > count_files.sh << 'EOF'
#!/bin/bash
for ext in txt sh log; do
    count=$(ls -1 *.$ext 2>/dev/null | wc -l)
    echo "File .$ext: $count"
done
EOF
chmod +x count_files.sh
./count_files.sh

# Task 3
./analyze_log.sh
grep -c "ERROR" company_app.log

# Task 4
ENV=production ./deploy.sh
```

## Lưu ý quan trọng
- Luôn bắt đầu script bằng `#!/bin/bash`.
- Dùng `"$VAR"` (có dấu ngoặc kép) để tránh lỗi khi biến chứa khoảng trắng.
- Kiểm tra lỗi bằng `$?` — `0` là thành công.
- DevOps hay dùng `set -e` ở đầu script để dừng ngay khi có lệnh lỗi.

---

# Bài 16: Docker Trên Ubuntu

> ⏱️ Thời gian: ~60 phút (15p lý thuyết + 30p thực hành + 15p task)
> 
> ⚠️ **Yêu cầu:** Bạn cần cài Docker Engine trước khi làm bài này.

## Mục tiêu
- Dùng Docker từ trong Ubuntu
- Chạy container, xem log, exec vào container
- Hiểu volume mount
- DevOps containerize ứng dụng bằng Docker

## Chuẩn bị: Cài Docker Engine (nếu chưa cài)

Trên Ubuntu, bạn có thể cài Docker Engine bằng:
```bash
sudo apt update
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker hoang
```

> Sau khi thêm user vào nhóm `docker`, bạn cần **đăng xuất và đăng nhập lại** (hoặc chạy `newgrp docker`) để quyền có hiệu lực.

Kiểm tra:
```bash
docker --version
docker ps
```

## Lý thuyết

Docker cho phép đóng gói ứng dụng vào **container** — chạy đồng nhất trên mọi môi trường.

| Lệnh | Ý nghĩa |
|------|---------|
| `docker ps` | Xem container đang chạy |
| `docker ps -a` | Xem tất cả container (cả đã dừng) |
| `docker run <image>` | Chạy container từ image |
| `docker run -d <image>` | Chạy nền (detached) |
| `docker logs <container>` | Xem log container |
| `docker exec -it <container> bash` | Vào trong container |
| `docker stop <container>` | Dừng container |
| `docker rm <container>` | Xóa container |
| `docker images` | Xem image đang có |
| `docker rmi <image>` | Xóa image |

> 🎯 DevOps thực tế: `docker build -t myapp .`, `docker run -d -p 80:80 myapp`, `docker logs -f myapp`.

## Thực hành từng bước

### Bước 1: Kiểm tra Docker
```bash
docker --version
docker ps
```

Nếu báo lỗi permission:
```bash
sudo usermod -aG docker hoang
```

> Cần đăng xuất rồi đăng nhập lại để nhóm `docker` có hiệu lực. Hoặc chạy `newgrp docker`.

### Bước 2: Chạy container hello-world
```bash
docker run hello-world
```

Output:
```
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

### Bước 3: Chạy container nền (nginx)
```bash
docker run -d --name mynginx -p 8080:80 nginx
```

Giải thích:
- `-d` = chạy nền
- `--name mynginx` = đặt tên container
- `-p 8080:80` = map port 8080 máy bạn → port 80 container

Kiểm tra:
```bash
docker ps
```

Output có `mynginx` đang chạy.

### Bước 4: Test nginx
```bash
curl http://localhost:8080
```

Output: HTML của trang welcome nginx.

### Bước 5: Xem log container
```bash
docker logs mynginx
```

Theo dõi realtime:
```bash
docker logs -f mynginx
```

Thoát: `Ctrl + C`.

### Bước 6: Vào trong container
```bash
docker exec -it mynginx bash
```

Bên trong container:
```bash
whoami
hostname
ls /usr/share/nginx/html/
exit
```

### Bước 7: Dừng và xóa container
```bash
docker stop mynginx
docker rm mynginx
docker ps -a
```

### Bước 8: Volume mount — chia sẻ file giữa host và container
```bash
cd ~/Test
echo "<h1>Hello from Ubuntu</h1>" > index.html

docker run -d --name mynginx2 -p 8080:80 -v $(pwd)/index.html:/usr/share/nginx/html/index.html nginx
```

> `-v <host>:<container>` = mount file/thư mục từ host vào container.

Test:
```bash
curl http://localhost:8080
```

Output:
```
<h1>Hello from Ubuntu</h1>
```

Dọn dẹp:
```bash
docker stop mynginx2
docker rm mynginx2
```

### Bước 9: Xem image đang có
```bash
docker images
```

Xóa image không cần:
```bash
docker rmi hello-world nginx
```

## Task cuối bài (15 phút)

1. Chạy container `httpd` (Apache) tên `myapache`, port 8081:80, chạy nền.
2. Kiểm tra `myapache` đang chạy bằng `docker ps`.
3. Vào trong `myapache` và xem file ở `/usr/local/apache2/htdocs/`.
4. Dừng và xóa `myapache`, kiểm tra lại `docker ps -a`.

## Đáp án / Gợi ý

```bash
# Task 1
docker run -d --name myapache -p 8081:80 httpd

# Task 2
docker ps

# Task 3
docker exec -it myapache bash
ls /usr/local/apache2/htdocs/
cat /usr/local/apache2/htdocs/index.html
exit

# Task 4
docker stop myapache
docker rm myapache
docker ps -a
```

## Lưu ý quan trọng
- Container là **stateless** — dữ liệu trong container mất khi xóa. Dùng **volume** để lưu dữ liệu lâu dài.
- `docker exec -it` giống SSH vào server nhỏ bên trong container.
- DevOps hay viết `Dockerfile` để build image. Ở đây bạn dùng image có sẵn để làm quen.
- Trên Ubuntu thật, Docker chạy native với kernel Linux — không qua lớp abstraction.

---

# Bài 17: Tự Động Hóa Deploy Giả Lập

> ⏱️ Thời gian: ~60 phút (15p lý thuyết + 30p thực hành + 15p task)

## Mục tiêu
- Viết script mô phỏng pipeline CI/CD: test → build → deploy
- Dùng file marker để kiểm tra trạng thái
- Thêm rollback đơn giản
- Hiểu mindset automate của DevOps

## Lý thuyết

Một pipeline CI/CD đơn giản gồm 3 stage:
1. **Test** — kiểm tra cơ bản (ví dụ: syntax check)
2. **Build** — tạo artifact (file nén, image)
3. **Deploy** — copy lên target, restart service

DevOps luôn tự động hóa để tránh lỗi con người.

## Thực hành từng bước

### Bước 1: Tạo cấu trúc giả lập
```bash
cd ~/Test
mkdir -p deploy_project/{src,build,deploy}
```

Tạo file nguồn giả lập:
```bash
cat > deploy_project/src/app.sh << 'EOF'
#!/bin/bash
echo "MyApp version 1.0 running"
EOF
```

### Bước 2: Viết script pipeline
```bash
cat > deploy_project/pipeline.sh << 'EOF'
#!/bin/bash

set -e  # Dung ngay neu co loi

PROJECT_DIR="/home/hoang/Test/deploy_project"
BUILD_DIR="${PROJECT_DIR}/build"
DEPLOY_DIR="${PROJECT_DIR}/deploy"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

log() {
    echo "[$(date '+%H:%M:%S')] $1"
}

# === STAGE 1: TEST ===
log "=== STAGE 1: TEST ==="
if [ ! -f "${PROJECT_DIR}/src/app.sh" ]; then
    log "ERROR: app.sh khong ton tai"
    exit 1
fi

# Kiem tra syntax bang bash -n
bash -n "${PROJECT_DIR}/src/app.sh"
log "TEST: Syntax OK"

# === STAGE 2: BUILD ===
log "=== STAGE 2: BUILD ==="
mkdir -p "$BUILD_DIR"
ARTIFACT="${BUILD_DIR}/app_${TIMESTAMP}.tar.gz"
tar -czf "$ARTIFACT" -C "${PROJECT_DIR}/src" .
log "BUILD: Artifact created at $ARTIFACT"

# === STAGE 3: DEPLOY ===
log "=== STAGE 3: DEPLOY ==="
mkdir -p "$DEPLOY_DIR"
cp "$ARTIFACT" "${DEPLOY_DIR}/current.tar.gz"

# Giai nen deploy
cd "$DEPLOY_DIR"
tar -xzf current.tar.gz
chmod +x app.sh

# Luu version hien tai
echo "$TIMESTAMP" > "${DEPLOY_DIR}/version.txt"
log "DEPLOY: Version $TIMESTAMP deployed"

log "=== PIPELINE SUCCESS ==="
EOF
```

Cho phép chạy:
```bash
chmod +x deploy_project/pipeline.sh
```

### Bước 3: Chạy pipeline
```bash
cd ~/Test
./deploy_project/pipeline.sh
```

Output mong đợi:
```
[08:30:00] === STAGE 1: TEST ===
[08:30:00] TEST: Syntax OK
[08:30:00] === STAGE 2: BUILD ===
[08:30:00] BUILD: Artifact created at ...
[08:30:00] === STAGE 3: DEPLOY ===
[08:30:00] DEPLOY: Version ... deployed
[08:30:00] === PIPELINE SUCCESS ===
```

Kiểm tra:
```bash
ls deploy_project/build/
cat deploy_project/deploy/version.txt
ls deploy_project/deploy/
```

### Bước 4: Script rollback
```bash
cat > deploy_project/rollback.sh << 'EOF'
#!/bin/bash

DEPLOY_DIR="/home/hoang/Test/deploy_project/deploy"
BUILD_DIR="/home/hoang/Test/deploy_project/build"

echo "Current version: $(cat ${DEPLOY_DIR}/version.txt)"
echo "Available artifacts:"
ls -lt "$BUILD_DIR" | head -5

# Rollback ve artifact moi nhat (tru cai hien tai)
LATEST=$(ls -t "$BUILD_DIR" | head -2 | tail -1)
if [ -n "$LATEST" ]; then
    cp "${BUILD_DIR}/${LATEST}" "${DEPLOY_DIR}/current.tar.gz"
    cd "$DEPLOY_DIR"
    tar -xzf current.tar.gz
    echo "Rollback to ${LATEST} done"
else
    echo "No previous artifact found"
fi
EOF

chmod +x deploy_project/rollback.sh
```

Chạy pipeline lần nữa để có thêm artifact, rồi rollback:
```bash
./deploy_project/pipeline.sh
./deploy_project/rollback.sh
```

### Bước 5: Tích hợp `deploy.sh` vào pipeline — Storyline

Nhớ `deploy.sh` bạn đã xây dựng từ Bài 1, 4, 6, 15? Giờ ta đưa nó vào pipeline thực tế.

```bash
cat > deploy_project/src/deploy.sh << 'EOF'
#!/bin/bash

set -e

ENV="${ENV:-staging}"
VERSION="${VERSION:-1.0.0}"

echo "[$(date)] Deploying version $VERSION to $ENV"
echo "[$(date)] Deploy completed successfully"
EOF

chmod +x deploy_project/src/deploy.sh
```

Sửa pipeline để chạy `deploy.sh` trong stage deploy:
```bash
cat > deploy_project/pipeline.sh << 'EOF'
#!/bin/bash

set -e

PROJECT_DIR="/home/hoang/Test/deploy_project"
BUILD_DIR="${PROJECT_DIR}/build"
DEPLOY_DIR="${PROJECT_DIR}/deploy"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

log() {
    echo "[$(date '+%H:%M:%S')] $1"
}

# === STAGE 1: TEST ===
log "=== STAGE 1: TEST ==="
for f in app.sh deploy.sh; do
    bash -n "${PROJECT_DIR}/src/$f"
    log "TEST: $f OK"
done

# === STAGE 2: BUILD ===
log "=== STAGE 2: BUILD ==="
mkdir -p "$BUILD_DIR"
ARTIFACT="${BUILD_DIR}/release_${TIMESTAMP}.tar.gz"
tar -czf "$ARTIFACT" -C "${PROJECT_DIR}/src" .
log "BUILD: Artifact created at $ARTIFACT"

# === STAGE 3: DEPLOY ===
log "=== STAGE 3: DEPLOY ==="
mkdir -p "$DEPLOY_DIR"
cp "$ARTIFACT" "${DEPLOY_DIR}/current.tar.gz"
cd "$DEPLOY_DIR"
tar -xzf current.tar.gz
chmod +x *.sh
log "DEPLOY: Running deploy.sh..."
ENV=production VERSION="${TIMESTAMP}" ./deploy.sh

echo "$TIMESTAMP" > "${DEPLOY_DIR}/version.txt"
log "DEPLOY: Version $TIMESTAMP deployed"
log "=== PIPELINE SUCCESS ==="
EOF

chmod +x deploy_project/pipeline.sh
./deploy_project/pipeline.sh
```

> 🎯 **Storyline:** Pipeline giờ đã tích hợp `deploy.sh` bạn viết từ đầu khóa học. Đó là cách CI/CD thực tế hoạt động — pipeline gọi script deploy đã được test kỹ.

## Task cuối bài (15 phút)

1. Sửa `deploy_project/src/app.sh` đổi version thành `1.1`, chạy lại pipeline.
2. Kiểm tra `deploy_project/build/` có ít nhất 2 file `.tar.gz`.
3. Viết script `health_deploy.sh` kiểm tra xem file `app.sh` có tồn tại trong `deploy/` không. Nếu có, in "Deploy OK", ngược lại in "Deploy FAILED".
4. Chạy `health_deploy.sh`.

## Đáp án / Gợi ý

```bash
cd ~/Test

# Task 1
cat > deploy_project/src/app.sh << 'EOF'
#!/bin/bash
echo "MyApp version 1.1 running"
EOF
./deploy_project/pipeline.sh

# Task 2
ls -lh deploy_project/build/

# Task 3
cat > health_deploy.sh << 'EOF'
#!/bin/bash
if [ -f "/home/hoang/Test/deploy_project/deploy/app.sh" ]; then
    echo "Deploy OK"
else
    echo "Deploy FAILED"
fi
EOF
chmod +x health_deploy.sh

# Task 4
./health_deploy.sh
```

## Lưu ý quan trọng
- `set -e` là bạn thân của DevOps — dừng ngay khi lệnh fail, không để lỗi lan ra.
- Pipeline thực tế dùng Jenkins/GitLab CI/GitHub Actions. Nhưng cuối cùng, runner vẫn chạy shell script trên server.
- Artifact = kết quả build (file nén, binary, image). Luôn lưu artifact để rollback.
- `bash -n` chỉ kiểm tra syntax, không chạy script.

---

# Bài 18: Tổng Hợp & Ôn Tập Phỏng Vấn

> ⏱️ Thời gian: ~60 phút (20p lý thuyết + 30p thực hành + 10p tổng kết)

## Mục tiêu
- Ôn tập toàn bộ lệnh quan trọng
- Giải quyết scenario thực tế
- Chuẩn bị câu trả lời phỏng vấn Linux/DevOps

## Lý thuyết

Những câu hỏi phỏng vấn Linux cho DevOps thường gặp:

1. **"Server chậm, bạn kiểm tra gì đầu tiên?"**
   - `top`, `htop` → CPU/RAM
   - `df -h` → disk full?
   - `iostat` → I/O bottleneck?

2. **"App không chạy, bạn debug thế nào?"**
   - `systemctl status <app>`
   - `journalctl -u <app> -f`
   - `cat /var/log/<app>/error.log`
   - `ss -tulpn | grep <port>`

3. **"Bạn deploy code mới, cần rollback?"**
   - Có artifact cũ → restore
   - `systemctl restart` hoặc `docker rollback`

4. **"Tại sao `chmod 777` nguy hiểm?"**
   - Ai cũng sửa được → security risk.
   - Production thường dùng `644` file, `755` script, `700` key.

## Thực hành từng bước — Scenario 1: Server chậm

### Bước 1: Giả lập server chậm (tạo file lớn)
```bash
cd ~/Test

dd if=/dev/zero of=load_test bs=1M count=200 &
```

> `&` = chạy nền. Process này tạo file lớn, tiêu tốn I/O.

### Bước 2: Kiểm tra hệ thống
```bash
# CPU/Memory
top -bn1 | head -20

# Disk
df -h

# Process nào đang "ngốn" tài nguyên
ps aux --sort=-%cpu | head -10
```

### Bước 3: Tìm và dừng process gây tải
```bash
pgrep dd
kill <PID_cua_dd>
```

Dọn file:
```bash
rm load_test
```

## Scenario 2: App crash, cần xem log

### Bước 4: Tạo log lỗi giả lập
```bash
cat > /tmp/myapp_error.log << 'EOF'
2025-07-16 08:00:00 ERROR Database connection timeout
2025-07-16 08:00:05 ERROR Failed to fetch user data
2025-07-16 08:00:10 INFO Retrying connection...
2025-07-16 08:00:15 WARN High memory usage
2025-07-16 08:00:20 ERROR Database connection timeout
EOF
```

### Bước 5: Phân tích log
```bash
# Dem so loi ERROR
grep "ERROR" /tmp/myapp_error.log | wc -l

# Xem loi xuat hien may lan, loai nao
grep "ERROR" /tmp/myapp_error.log | awk '{print $4,$5,$6}' | sort | uniq -c

# Theo doi log realtime
tail -f /tmp/myapp_error.log &
echo "2025-07-16 08:00:25 ERROR New crash" >> /tmp/myapp_error.log
kill %1
```

## Scenario 3: Kiểm tra port và kết nối

### Bước 6: Xem port đang mở
```bash
ss -tulpn | head -20
```

### Bước 7: Test kết nối đến một dịch vụ
```bash
curl -I https://google.com
ping -c 3 8.8.8.8
```

## Task cuối bài (10 phút)

Hãy viết 1 script `server_audit.sh` tổng hợp:
1. In tên máy (`hostname`).
2. In ngày giờ hiện tại.
3. In disk usage của `/`.
4. In top 3 process dùng nhiều CPU nhất.
5. In số lượng file `.log` trong `/var/log` (nếu có quyền).
6. **(Storyline)** Phân tích `company_app.log`: tổng dòng, số ERROR, số WARN.
7. Ghi tất cả vào file `audit_$(date +%Y%m%d).txt`.

## Đáp án / Gợi ý

```bash
cd ~/Test

cat > server_audit.sh << 'EOF'
#!/bin/bash

OUTPUT="audit_$(date +%Y%m%d).txt"
APP_LOG="/home/hoang/Test/company_app.log"

{
    echo "=== SERVER AUDIT ==="
    echo "Hostname: $(hostname)"
    echo "Time: $(date)"
    echo ""
    echo "=== DISK USAGE ==="
    df -h /
    echo ""
    echo "=== TOP 3 CPU PROCESSES ==="
    ps aux --sort=-%cpu | head -4
    echo ""
    echo "=== LOG FILE COUNT ==="
    find /var/log -name "*.log" 2>/dev/null | wc -l
    echo ""
    echo "=== COMPANY APP LOG ANALYSIS ==="
    if [ -f "$APP_LOG" ]; then
        echo "File: $APP_LOG"
        echo "Total lines: $(wc -l < "$APP_LOG")"
        echo "ERROR count: $(grep -c "ERROR" "$APP_LOG")"
        echo "WARN count: $(grep -c "WARN" "$APP_LOG")"
        echo "INFO count: $(grep -c "INFO" "$APP_LOG")"
    else
        echo "company_app.log not found"
    fi
    echo "=== END ==="
} > "$OUTPUT"

echo "Audit saved to $OUTPUT"
cat "$OUTPUT"
EOF

chmod +x server_audit.sh
./server_audit.sh
```

## Lưu ý quan trọng
- Phỏng vấn DevOps không yêu cầu nhớ hết lệnh, nhưng cần biết **debug flow**: từ symptom → tool phù hợp → root cause.
- Luôn nhớ: `top` (resource), `df` (disk), `ps` (process), `ss` (port), `journalctl` (log).
- Script tự động hóa là điểm cộng lớn trong phỏng vấn.

---

## ✅ Kết Thúc Toàn Bộ Lộ Trình

Chúc mừng! Bạn đã hoàn thành 19 bài học Linux cho DevOps (7 bài Giai đoạn 1 + 8 bài Giai đoạn 2 + 4 bài Giai đoạn 3).

### Bạn biết được:
- Quản lý file, quyền, process
- Cài phần mềm, quản lý service
- Network, SSH, cron, log
- Script tự động hóa
- Docker cơ bản
- Mindset CI/CD pipeline

### Hành trình 2 file "truyền kỳ" của bạn:
- **`deploy.sh`**: Từ file trống (Bài 1) → có biến môi trường (Bài 4) → sửa bằng vim (Bài 6) → có function & log (Bài 15) → tích hợp pipeline (Bài 17)
- **`company_app.log`**: Từ 6 dòng log mẫu (Bài 5) → sửa typo (Bài 6) → theo dõi realtime + logrotate (Bài 13) → chown + find (Bài 14) → script phân tích (Bài 15) → audit (Bài 18)

> 💡 Đây chính xác là cách file "sống" trong production — được tạo, sửa, giám sát, phân tích qua nhiều ngày, nhiều người.

### Bước tiếp theo:
1. Mở file `99-cheat-sheet.md` để tra cứu nhanh.
2. Thực hành lại các scenario trong bài 18 mỗi tuần 1 lần.
3. Học tiếp: Docker nâng cao, Kubernetes, Terraform, Ansible.

**Keep learning, keep automating!** 🚀
