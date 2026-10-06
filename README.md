# Báo cáo Bài 3: Cấu hình tường lửa UFW và chẩn đoán cổng mạng

## 1. Mục tiêu & Yêu cầu
- **Mục tiêu:**
  - Sử dụng công cụ tường lửa **UFW** (Uncomplicated Firewall) để bảo vệ máy chủ Linux.
  - Thiết lập chính sách mặc định và các quy tắc (rules) chặn/mở cổng mạng phục vụ việc deploy ứng dụng một cách an toàn.
  - Sử dụng các công cụ chẩn đoán mạng (`ufw status verbose`, `ss -tlnp`, `curl`) để kiểm tra trạng thái hoạt động và cổng kết nối.
- **Bối cảnh & Ràng buộc:**
  - Cấu hình chính sách mặc định: Chặn toàn bộ kết nối đi vào (`deny incoming`), cho phép toàn bộ kết nối đi ra (`allow outgoing`).
  - Mở cổng SSH (`22/tcp`) để đảm bảo không mất kết nối quản trị từ xa.
  - Mở cổng ứng dụng Web (`8080/tcp`).
  - Kích hoạt tường lửa UFW và xác thực trạng thái hoạt động.

---

## 2. Các bước triển khai

### Bước 1: Thiết lập chính sách mặc định của tường lửa
Thiết lập chính sách bảo mật cơ bản: Chặn tất cả lưu lượng đi vào và cho phép lưu lượng đi ra:
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```
**Kết quả thực thi:**
```text
Default incoming policy changed to 'deny'
(be sure to update your rules accordingly)
Default outgoing policy changed to 'allow'
(be sure to update your rules accordingly)
```

---

### Bước 2: Thêm quy tắc cho phép các cổng cần thiết
Mở cổng SSH (`22/tcp`) để duy trì kết nối quản trị:
```bash
sudo ufw allow 22/tcp
```

Mở cổng ứng dụng Web (`8080/tcp`):
```bash
sudo ufw allow 8080/tcp
```
**Kết quả thực thi:**
```text
Rules updated
Rules updated (v6)
```

---

### Bước 3: Kích hoạt tường lửa UFW
Kích hoạt UFW và cho phép khởi động cùng hệ thống:
```bash
sudo ufw enable
```
*(Nếu được hỏi xác nhận có thể làm ngắt kết nối SSH, chọn `y` và nhấn Enter)*:
```text
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup
```

---

## 3. Kiểm tra và Chẩn đoán mạng

### 3.1. Kiểm tra trạng thái chi tiết của tường lửa (`ufw status verbose`)
```bash
sudo ufw status verbose
```

**Kết quả thực tế trên hệ thống:**
```text
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    Anywhere                  
8080/tcp                   ALLOW IN    Anywhere                  
22/tcp (v6)                ALLOW IN    Anywhere (v6)             
8080/tcp (v6)              ALLOW IN    Anywhere (v6)             
```

#### Phân tích kết quả:
- **`Status: active`**: Tường lửa đang hoạt động và đang bảo vệ hệ thống.
- **`Default: deny (incoming), allow (outgoing)`**: Mặc định từ chối toàn bộ truy cập đi vào trừ khi có rule cho phép; chiều đi ra được mở hoàn toàn.
- **`22/tcp ALLOW IN Anywhere`**: Cổng quản trị SSH được mở từ mọi nguồn IPv4 và IPv6.
- **`8080/tcp ALLOW IN Anywhere`**: Cổng dịch vụ Web 8080 được mở từ mọi nguồn IPv4 và IPv6.

---

### 3.2. Kiểm tra các cổng đang lắng nghe trên máy chủ (`ss -tlnp`)
Lệnh `ss` (Socket Statistics) cho phép liệt kê các socket đang mở và ứng dụng quản lý cổng đó:
```bash
ss -tlnp
```

**Kết quả kiểm tra thực tế:**
```text
State  Recv-Q Send-Q  Local Address:Port Peer Address:PortProcess                                  
LISTEN 0      4096    127.0.0.53%lo:53        0.0.0.0:*    users:(("systemd-resolve",pid=84,fd=17))
LISTEN 0      4096       127.0.0.54:53        0.0.0.0:*    users:(("systemd-resolve",pid=84,fd=19))
LISTEN 0      1000   10.255.255.254:53        0.0.0.0:*                                            
```

#### Giải thích tham số lệnh `ss -tlnp`:
- `-t`: Chỉ hiển thị giao thức **TCP**.
- `-l`: Chỉ lọc các cổng đang ở trạng thái **LISTEN** (đang lắng nghe kết nối).
- `-n`: Hiển thị số hiệu cổng thay vì cố gắng phân giải thành tên dịch vụ.
- `-p`: Hiển thị Process / PID của tiến trình đang chiếm giữ socket.

---

## 4. Kết luận
- Đã thiết lập thành công mô hình phòng thủ nhiều lớp cho máy chủ:
  1. Chính sách mặc định thắt chặt (`deny incoming`).
  2. Mở cổng có kiểm soát cho SSH (`port 22`) và ứng dụng Web (`port 8080`).
  3. Tường lửa hoạt động ổn định và sẵn sàng cho việc deploy ứng dụng một cách an toàn.
