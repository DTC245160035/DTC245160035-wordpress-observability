# DTC245160035 - Website quảng bá sản phẩm WordPress

Hệ thống website quảng bá sản phẩm sử dụng WordPress, MySQL, phpMyAdmin và Nginx Reverse Proxy. Toàn bộ dịch vụ được triển khai bằng Docker Compose trên máy ảo Ubuntu Linux.

## 1. Thông tin sinh viên

- Họ và tên: Hoàng Việt
- Mã sinh viên: DTC245160035
- Chủ đề: Website quảng bá sản phẩm sử dụng WordPress
- Nền tảng: Máy ảo Ubuntu Linux
- Phương thức triển khai: Docker Compose

## 2. GitHub Repository

Repository của dự án:

```text
https://github.com/DTC245160035/DTC245160035-wordpress-observability
```

Repository chứa:

- Mã nguồn và cấu hình Docker Compose.
- Cấu hình Nginx Reverse Proxy.
- Cấu hình giám sát Prometheus và Grafana.
- Cấu hình log tập trung Loki và Promtail.
- README hướng dẫn triển khai hệ thống.

## 3. Các thành phần của hệ thống

Các thành phần hiện tại:

- WordPress: xây dựng website quảng bá sản phẩm.
- MySQL 8.4: lưu trữ dữ liệu WordPress.
- phpMyAdmin: quản trị cơ sở dữ liệu MySQL.
- Nginx: Reverse Proxy và bổ sung security headers.

Các thành phần sẽ được bổ sung trong các commit tiếp theo:

- Prometheus: thu thập metrics.
- Grafana: hiển thị dashboard giám sát.
- cAdvisor: cung cấp metrics container.
- Nginx Exporter: cung cấp metrics Nginx.
- MySQL Exporter: cung cấp metrics MySQL.
- Loki: lưu trữ log tập trung.
- Promtail: thu thập log container.

## 4. Kiến trúc hệ thống

Luồng truy cập website:

```text
Client
   |
   v
Nginx Reverse Proxy
   |
   v
WordPress
   |
   v
MySQL
```

phpMyAdmin kết nối trực tiếp với MySQL qua Docker backend network.

```text
Mac
 |
 | SSH tunnel
 v
phpMyAdmin
 |
 v
MySQL
```

Nginx là điểm truy cập chính của website. WordPress và MySQL không công khai trực tiếp ra bên ngoài.

## 5. Cấu trúc thư mục

```text
DTC245160035-wordpress-observability/
├── compose.yaml
├── README.md
├── .env.example
├── .gitignore
├── nginx/
│   ├── conf.d/
│   │   └── default.conf
│   └── certs/
├── prometheus/
├── grafana/
│   └── provisioning/
├── loki/
├── promtail/
├── scripts/
└── docs/
    └── screenshots/
```

## 6. Yêu cầu môi trường

Máy triển khai cần có:

- Ubuntu Linux hoặc hệ điều hành hỗ trợ Docker.
- Docker.
- Docker Compose.
- Git.
- OpenSSL.
- Tối thiểu 4 GB RAM.

Kiểm tra công cụ:

```bash
git --version
docker --version
docker compose version
openssl version
```

## 7. Cấu hình biến môi trường

Sao chép file cấu hình mẫu:

```bash
cp .env.example .env
```

Mở file `.env`:

```bash
nano .env
```

Cấu hình các biến:

```dotenv
MYSQL_DATABASE=wordpress
MYSQL_USER=wordpress_user
MYSQL_PASSWORD=YOUR_STRONG_MYSQL_PASSWORD
MYSQL_ROOT_PASSWORD=YOUR_STRONG_ROOT_PASSWORD

WORDPRESS_DB_HOST=mysql:3306
WORDPRESS_DB_NAME=wordpress
WORDPRESS_DB_USER=wordpress_user
WORDPRESS_DB_PASSWORD=YOUR_STRONG_MYSQL_PASSWORD
WORDPRESS_TABLE_PREFIX=wp_
```

Lưu ý:

- Thay các mật khẩu mẫu bằng mật khẩu thực tế.
- `MYSQL_PASSWORD` và `WORDPRESS_DB_PASSWORD` phải giống nhau.
- Không commit file `.env` lên GitHub.
- Chỉ file `.env.example` được lưu trong repository.

## 8. Khởi động hệ thống

Di chuyển vào thư mục dự án:

```bash
cd ~/DTC245160035-wordpress-observability
```

Kiểm tra cấu hình Docker Compose:

```bash
sudo docker compose config --quiet
```

Tải các Docker image:

```bash
sudo docker compose pull
```

Khởi động hệ thống:

```bash
sudo docker compose up -d
```

Kiểm tra trạng thái:

```bash
sudo docker compose ps
```

Các container cần hoạt động:

```text
dtc245160035-nginx
dtc245160035-wordpress
dtc245160035-mysql
dtc245160035-phpmyadmin
```

MySQL phải có trạng thái `healthy`.

## 9. Dừng và khởi động lại hệ thống

Dừng container nhưng giữ nguyên dữ liệu:

```bash
sudo docker compose down
```

Khởi động lại:

```bash
sudo docker compose up -d
```

Xem log:

```bash
sudo docker compose logs --tail=100
```

Không sử dụng `docker compose down -v` nếu muốn giữ dữ liệu WordPress và MySQL.

## 10. Địa chỉ truy cập dịch vụ

### 10.1. Website WordPress

Website được truy cập thông qua Nginx Reverse Proxy:

```text
http://172.172.15.82
```

Trang quản trị WordPress:

```text
http://172.172.15.82/wp-admin/
```

Nếu địa chỉ IP máy ảo thay đổi, kiểm tra bằng:

```bash
hostname -I
```

Sau đó sử dụng địa chỉ IPv4 của card mạng `ens160`.

### 10.2. phpMyAdmin

phpMyAdmin chỉ lắng nghe trên localhost của máy ảo:

```text
127.0.0.1:8081
```

Để truy cập từ máy Mac, mở Terminal trên Mac và tạo SSH tunnel:

```bash
ssh -N \
  -L 18081:127.0.0.1:8081 \
  hnqv3526@172.172.15.82
```

Giữ cửa sổ Terminal này hoạt động và mở:

```text
http://127.0.0.1:18081
```

phpMyAdmin sử dụng tài khoản MySQL được khai báo trong `.env`:

```text
Username: giá trị MYSQL_USER
Password: giá trị MYSQL_PASSWORD
```

Không sử dụng mật khẩu thật trong README.

## 11. Nginx Reverse Proxy

Nginx nhận yêu cầu tại cổng 80 và chuyển tiếp đến WordPress qua Docker frontend network.

Luồng xử lý:

```text
Client -> Nginx:80 -> WordPress:80
```

Cấu hình chuyển tiếp chính:

```nginx
location / {
    proxy_pass http://wordpress:80;
    proxy_http_version 1.1;

    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}
```

Kiểm tra website qua Nginx:

```bash
curl -I http://172.172.15.82
```

Kiểm tra cú pháp Nginx:

```bash
sudo docker exec dtc245160035-nginx nginx -t
```

Kết quả hợp lệ:

```text
syntax is ok
test is successful
```

## 12. Security Headers

Nginx áp dụng các HTTP security header cơ bản:

```nginx
add_header X-Content-Type-Options "nosniff" always;
add_header X-Frame-Options "SAMEORIGIN" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
add_header Permissions-Policy "geolocation=(), camera=(), microphone=()" always;
```

Kiểm tra:

```bash
curl -I http://172.172.15.82
```

Kết quả phải có:

```text
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: geolocation=(), camera=(), microphone=()
```

## 13. Docker Network

Hệ thống sử dụng hai Docker network.

### Frontend network

Các container tham gia:

- Nginx.
- WordPress.
- phpMyAdmin.

Nginx sử dụng mạng frontend để kết nối WordPress.

### Backend network

Các container tham gia:

- WordPress.
- MySQL.
- phpMyAdmin.

Cấu hình mạng backend:

```yaml
backend:
  name: dtc245160035-backend
  driver: bridge
  internal: true
```

MySQL chỉ tham gia backend network và không publish cổng `3306` ra máy host.

WordPress kết nối MySQL bằng địa chỉ nội bộ:

```text
mysql:3306
```

## 14. Các biện pháp bảo mật hiện tại

Hệ thống hiện áp dụng:

1. MySQL không công khai cổng `3306` ra máy host.
2. Backend network được đặt ở chế độ `internal`.
3. phpMyAdmin chỉ bind vào `127.0.0.1`.
4. WordPress chỉ được truy cập thông qua Nginx.
5. File `.env` được loại khỏi Git bằng `.gitignore`.
6. Các container sử dụng `no-new-privileges`.
7. Nginx áp dụng các HTTP security header cơ bản.
8. Mật khẩu không được ghi trực tiếp trong `compose.yaml`.

## 15. Kiểm tra hệ thống

Kiểm tra container:

```bash
sudo docker compose ps
```

Kiểm tra Nginx:

```bash
sudo docker exec dtc245160035-nginx nginx -t
```

Kiểm tra website:

```bash
curl -I http://172.172.15.82
```

Kiểm tra các network:

```bash
sudo docker network inspect dtc245160035-frontend
sudo docker network inspect dtc245160035-backend
```

Kiểm tra `.env` không bị Git theo dõi:

```bash
git check-ignore -v .env
```

## 16. Tiến độ triển khai

- [x] WordPress và MySQL.
- [x] phpMyAdmin.
- [x] Nginx Reverse Proxy.
- [x] Security headers cơ bản.
- [x] Docker network isolation.
- [ ] Prometheus và Grafana.
- [ ] cAdvisor, Nginx Exporter và MySQL Exporter.
- [ ] Loki và Promtail.
- [ ] Truy vấn log bằng LogQL.
- [ ] Hoàn thiện hardening và báo cáo.

## Hệ thống giám sát Prometheus và Grafana

Hệ thống sử dụng Prometheus để thu thập metrics và Grafana để trực quan hóa dữ liệu.

### Các thành phần giám sát

| Thành phần | Chức năng | Cổng nội bộ |
|---|---|---:|
| Prometheus | Thu thập và lưu trữ metrics | 9090 |
| Grafana | Hiển thị dashboard giám sát | 3000 |
| cAdvisor | Thu thập metrics của container | 8080 |
| Nginx Exporter | Xuất metrics của Nginx | 9113 |
| MySQL Exporter | Xuất metrics của MySQL | 9104 |

Prometheus thu thập dữ liệu từ bốn job:

- `prometheus`: giám sát chính Prometheus.
- `cadvisor`: giám sát tài nguyên và hoạt động của container.
- `nginx`: giám sát số kết nối và trạng thái Nginx.
- `mysql`: giám sát trạng thái và kết nối MySQL.

### Truy cập giao diện giám sát

Prometheus và Grafana chỉ được ánh xạ vào địa chỉ `127.0.0.1` của máy ảo. Từ máy Mac, tạo SSH tunnel:

```bash
ssh -N \
  -o ExitOnForwardFailure=yes \
  -L 13000:127.0.0.1:3000 \
  -L 19090:127.0.0.1:9090 \
  hnqv3526@172.172.15.82

```

Sau đó truy cập:

- Prometheus: `http://127.0.0.1:19090`
- Prometheus Targets: `http://127.0.0.1:19090/targets`
- Grafana: `http://127.0.0.1:13000`

Tài khoản Grafana được cấu hình bằng các biến môi trường:

```text
GRAFANA_ADMIN_USER
GRAFANA_ADMIN_PASSWORD
```

Giá trị thật được lưu trong file `.env` và không được đưa lên GitHub.

### Dashboard Grafana

Dashboard `DTC245160035 - System Monitoring` gồm các panel:

1. Trạng thái các target Prometheus.
2. Mức sử dụng CPU của các container.
3. Số kết nối Nginx đang hoạt động.
4. Số kết nối MySQL đang hoạt động.

Các truy vấn PromQL chính:

```promql
up
```

```promql
sum by (container_label_com_docker_compose_service) (
  rate(container_cpu_usage_seconds_total{
    container_label_com_docker_compose_service!=""
  }[5m])
) * 100
```

```promql
nginx_connections_active
```

```promql
mysql_global_status_threads_connected
```

### Kiểm tra hệ thống giám sát

```bash
sudo docker compose ps
curl -s 'http://127.0.0.1:9090/api/v1/query?query=up'
```

Các target `prometheus`, `cadvisor`, `nginx` và `mysql` phải có giá trị `1`, nghĩa là đang hoạt động bình thường.
