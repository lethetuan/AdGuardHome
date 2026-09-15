# Hướng dẫn chi tiết cài đặt AdGuard Home trực tiếp (Native) trên Ubuntu

**Mô tả:** Tài liệu này hướng dẫn các bước cài đặt AdGuard Home bằng script chính thức từ nhà phát triển, giúp tối ưu hiệu năng mạng và cấp đủ quyền hoạt động cho máy chủ DNS, thay vì sử dụng bản Snap bị giới hạn.

---

## Bước 1: Giải phóng cổng 53 (Bắt buộc)

Hệ điều hành Ubuntu mặc định sử dụng dịch vụ `systemd-resolved` để quản lý phân giải tên miền nội bộ, dịch vụ này luôn chiếm giữ cổng `53`. Chúng ta cần vô hiệu hóa tính năng nghe (StubListener) của nó để nhường cổng 53 cho ứng dụng AdGuard Home.

Chạy lần lượt các lệnh sau trong Terminal (đảm bảo bạn có quyền sudo/root):

1. Tạo thư mục chứa file cấu hình mới:
   ```bash
   sudo mkdir -p /etc/systemd/resolved.conf.d
   ```
2. Tạo file cấu hình để tắt StubListener:
```bash
echo -e "[Resolve]\nDNSStubListener=no" | sudo tee /etc/systemd/resolved.conf.d/adguard.conf
```
3. Sao lưu cấu hình cũ và cập nhật lại file phân giải tên miền của hệ thống:
```bash
sudo mv /etc/resolv.conf /etc/resolv.conf.backup
sudo ln -s /run/systemd/resolve/resolv.conf /etc/resolv.conf
```
```bash
sudo systemctl restart systemd-resolved
```
Gõ lệnh 
```bash
sudo lsof -i :53
```
Nếu Terminal không trả về kết quả nào (không hiện ra danh sách tiến trình), nghĩa là cổng 53 đã hoàn toàn trống và sẵn sàng để cài đặt AdGuard Home.
<img width="986" height="252" alt="image" src="https://github.com/user-attachments/assets/5890e74b-155c-4912-8aae-690b41f5120c" />

4. Mở port mạng cho AdGuard Home, cần khá nhiều cổng 3000, 53, 80 để hoạt động:
```bash

sudo ufw allow 3000/tcp      # Dành cho bước thiết lập web ban đầu
sudo ufw allow 53/udp        # Phân giải DNS cơ bản (Bắt buộc)
sudo ufw allow 53/tcp        # Phân giải DNS cơ bản (Bắt buộc)
sudo ufw allow 80/tcp        # Truy cập trang quản trị web (Sau khi setup)
sudo ufw reload
sudo ufw status

```



## Bước 2: Tải và chạy Script cài đặt chính thức từ nhà phát hành. Chạy lệnh duy nhất dưới đây để tải về bản cài đặt mới nhất và tự động thiết lập:

```bash
curl -s -S -L https://raw.githubusercontent.com/AdguardTeam/AdGuardHome/master/scripts/install.sh | sh -s -- -v
```
<img width="1420" height="1022" alt="image" src="https://github.com/user-attachments/assets/e88f7deb-2e10-4568-bc94-6d25e9faabb3" />

Chờ lệnh chạy xong. Màn hình Terminal sẽ hiển thị dòng chữ thông báo quá trình cài đặt thành công và cung cấp cho bạn địa chỉ IP kèm cổng 3000 (Ví dụ: http://192.168.1.10:3000) để tiếp tục cấu hình trên trình duyệt.

## Bước 3: Thiết lập cấu hình ban đầu trên trình duyệt Web
Mở trình duyệt web trên máy tính của bạn.

Truy cập vào địa chỉ IP của server kèm cổng 3000 (ví dụ: http://<IP_Ubuntu>:3000).

Nhấn Get Started và làm theo các bước:

Giao diện quản lý (Admin Web Interface): Chọn "Listen Interfaces" là All interfaces và đổi cổng (Port) từ 3000 thành 80.

Máy chủ DNS (DNS Server): Chọn "Listen Interfaces" là All interfaces và giữ nguyên cổng 53.

Xác thực: Tạo Tên đăng nhập và Mật khẩu cho tài khoản quản trị (Admin).

Cách kiểm tra thành công:
Mở một tab mới trên trình duyệt, gõ trực tiếp địa chỉ IP của server (không cần :3000 nữa). Nếu trang đăng nhập của AdGuard Home hiện ra, bạn đã cấu hình thành công.
