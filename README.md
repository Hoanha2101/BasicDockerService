# 🚀 Docker Nginx Reverse Proxy with SSL (Self-signed)

Dự án này demo cách sử dụng **Docker Compose** để triển khai nhiều service (service-a, service-b, service-c) chạy sau một **Nginx reverse proxy**, có hỗ trợ **HTTPS (SSL/TLS)** với chứng chỉ tự ký (*self-signed certificate*).

---

## 📂 Cấu trúc thư mục

```bash
.
├── docker-compose.yml
├── nginx/
│   ├── nginx.conf
│   └── certs/
│       ├── selfsigned.crt
│       └── selfsigned.key
└── html/
    ├── service-a/
    ├── service-b/
    └── service-c/
```

---

## 🔐 Tạo SSL Certificate (Self-signed)

### Linux / macOS

```bash
openssl req -x509 -nodes -days 365 \
  -newkey rsa:2048 \
  -keyout ./nginx/certs/selfsigned.key \
  -out ./nginx/certs/selfsigned.crt \
  -subj "/C=US/ST=State/L=City/O=LocalDev/CN=localhost"
```

### Windows
1. Tải và cài [Win64 OpenSSL](https://slproweb.com/products/Win32OpenSSL.html).
2. Mở **PowerShell** hoặc **CMD** trong thư mục dự án:

```powershell
openssl req -x509 -nodes -days 365 `
  -newkey rsa:2048 `
  -keyout nginx\certs\selfsigned.key `
  -out nginx\certs\selfsigned.crt `
  -subj "/CN=localhost" `
  -config "C:\Program Files\OpenSSL-Win64\bin\openssl.cfg"
```

📝 **Sau khi tạo**, bạn sẽ có 2 file:
* `nginx/certs/selfsigned.key`
* `nginx/certs/selfsigned.crt`

---

## ⚙️ Chạy dự án

1. **Khởi động:**

```bash
docker-compose up -d
```

2. **Kiểm tra container:**

```bash
docker ps
```

3. **Truy cập:**
   * https://localhost/service-a
   * https://localhost/service-b
   * https://localhost/service-c

⚠️ **Vì dùng self-signed cert**, trình duyệt sẽ báo *"Not Secure"* hoặc yêu cầu bạn xác nhận trust certificate. Chỉ cần bấm *Continue* để truy cập.

---

## 🏗️ Sơ đồ kiến trúc

```
🌐 Browser (HTTPS)
         ↓
    ┌─────────────┐
    │    Nginx    │ ← Port 443 (HTTPS)
    │ Reverse     │   Port 80 (HTTP → redirect to HTTPS)
    │ Proxy       │
    └─────────────┘
         ↓
    ┌─────────────────────────────────┐
    │         Docker Network          │
    │  ┌─────────┐ ┌─────────┐ ┌─────────┐  │
    │  │Service-A│ │Service-B│ │Service-C│  │
    │  │Port 3001│ │Port 3002│ │Port 3003│  │
    │  └─────────┘ └─────────┘ └─────────┘  │
    └─────────────────────────────────┘
```

---

## 📌 Lưu ý

* Các service `service-a`, `service-b`, `service-c` thực tế chỉ chạy **nội bộ trong Docker**. Người dùng chỉ cần truy cập thông qua **nginx reverse proxy**.
* Nếu muốn **ẩn hoàn toàn các port 3001, 3002, 3003**, bạn có thể bỏ phần `ports:` của từng service trong `docker-compose.yml`. Khi đó, chỉ nginx proxy mới kết nối được đến các service này.

---

## 🛠️ Debug

**Xem log của nginx:**
```bash
docker logs nginx
```

**Xem log của service cụ thể:**
```bash
docker logs service-a
docker logs service-b  
docker logs service-c
```

**Restart nginx:**
```bash
docker-compose restart nginx
```

---

## 🔄 Dừng dự án

```bash
docker-compose down
```

---

## ✅ Kết quả

Bạn đã có một hệ thống Nginx reverse proxy đơn giản chạy bằng Docker, hỗ trợ HTTPS với self-signed SSL certificate. Tất cả traffic từ browser sẽ được mã hóa SSL và được route đến đúng service backend tương ứng.

---

## 🚀 Mở rộng

- **Production**: Thay thế self-signed certificate bằng certificate từ Let's Encrypt hoặc CA authority
- **Load Balancing**: Có thể scale nhiều instance của mỗi service và config nginx làm load balancer
- **Monitoring**: Thêm logging, metrics với Prometheus/Grafana
- **Security**: Thêm rate limiting, WAF rules trong nginx config