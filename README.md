# buiminhhoang_hcm-cntt1_session18_bai2

## PHẦN 1. PHÂN TÍCH

### So sánh sự đánh đổi giữa Cache (MB/GB) và Cookies (~4KB)

| Tiêu chí | Web Cache | Cookies |
| :--- | :--- | :--- |
| **Dung lượng lưu trữ** | Lớn (**hàng trăm MB đến vài GB** tùy cấu hình bộ nhớ đệm trình duyệt/ổ cứng). | Cực kỳ hạn chế (**~4KB** cho mỗi cookie/domain). |
| **Bản chất dữ liệu** | Tài nguyên tĩnh (Static Assets): hình ảnh sản phẩm, CSS, JavaScript, font chữ, file media. | Dữ liệu trạng thái (State): Session Token, JWT, User ID, cấu hình phiên làm việc ngắn hạn. |
| **Cơ chế gửi qua mạng** | Lưu trữ cục bộ tại máy khách; không đính kèm vào HTTP Request Header trừ khi cần revalidate (`ETag`, `If-Modified-Since`). | **Tự động gửi kèm trong Header (`Cookie: ...`)** của mọi HTTP Request gửi lên server. |
| **Sự đánh đổi (Trade-off)** | Giảm tải băng thông và tăng tốc tải trang, nhưng **không phù hợp** để lưu định danh bảo mật vì không tự động xác thực request lên server. | Kích thước nhỏ gọn giúp tối ưu độ trễ mạng, nhưng nếu nhồi nhét quá 4KB sẽ làm phình to HTTP Header, gây chậm đường truyền mạng (**Network Overhead**). |

---

## PHẦN 2. TRIỂN KHAI

### Cấu trúc mẩu tin Cookie lưu phiên làm việc (Set-Cookie Header)

Để chuyển từ **Session Cookie** (mất khi đóng trình duyệt) sang **Persistent Cookie** (duy trì trạng thái) và chặn truy cập trái phép từ JavaScript, Server cần trả về mẩu tin qua Response Header:

```http
Set-Cookie: session_token=abc123xyz_secure_payload; Max-Age=604800; Path=/; Secure; HttpOnly; SameSite=Lax
```
Chi tiết các thành phần:
session_token=...: Chuỗi token định danh phiên làm việc của shipper (hoặc tham chiếu giỏ hàng lưu trong CSDL server).


HttpOnly (Ràng buộc bảo mật cốt lõi): Cấm mã JavaScript phía Client truy cập vào Cookie qua document.cookie, triệt tiêu hoàn toàn nguy cơ tấn công XSS (Cross-Site Scripting) để đánh cắp token (Session Hijacking).

Secure: Bắt buộc Cookie chỉ được truyền qua kết nối mã hóa HTTPS, ngăn chặn lộ lọt token trên đường truyền mạng (tấn công Man-in-the-Middle).


## PHẦN 3. KIỂM THỬ
 Hướng dẫn thao tác Hard Reload để xóa Cache đệm khi giao diện không cập nhật
 Khi hệ thống cập nhật CSS/giao diện mới nhưng trình duyệt vẫn tải stylesheet cũ từ Disk Cache:

- Cách 1: Phím tắt trực tiếp (Nhanh nhất)
  Nhấn tổ hợp phím: Ctrl + F5 (hoặc Ctrl + Shift + R trên Windows / Linux).
  
  Trình duyệt sẽ bỏ qua toàn bộ Cache cục bộ và buộc gửi request mới lấy file CSS/JS trực tiếp từ Server.

- Cách 2: Thao tác qua Chrome DevTools (Triệt để nhất)
  Nhấn phím F12 (hoặc Ctrl + Shift + I) để mở giao diện Developer Tools.
  
  Nhấp chuột phải trực tiếp vào biểu tượng Nút Tải lại trang (Reload / ↻) ở góc trái thanh địa chỉ trình duyệt.
  
  Chọn dòng "Empty Cache and Hard Reload" (Xóa bộ nhớ đệm và tải lại cứng).
  
  Kiểm tra lại giao diện: Trình duyệt đã dọn sạch cache tài nguyên giao diện cũ và áp dụng màu sắc mới.
