# K26CNTT2_IT108_Session16_Ex12
BTVN
# 🏥 Bài 12: Xử Lý Khủng Hoảng Hệ Thống Web CarePulse Ngày Tiêm Chủng Diện Rộng

> **Môn học:** Nhập Môn Công Nghệ Thông Tin (IT108)  
> **Sinh viên thực hiện:** Lê Hồ Thanh Thảo  
> **Mã sinh viên:** N26DTCN030  
> **Lớp:** D26TXCN02-N (K26CNTT2)  
> **Hệ thống:** Cổng Điện Tử Tiêm Chủng CarePulse (`tiemchung.carepulse.vn`)

---

## 📌 PHẦN 1: CHẨN ĐOÁN & XỬ LÝ SỰ CỐ ĐƯỜNG TRUYỀN & MÁY CHỦ

### 1. Bản chất kỹ thuật của mã 500 và sự khác biệt với mã 404

* **Tại sao máy chủ trả về mã `500` thay vì `200`?**
  * Trong đợt quá tải với 150,000 người dùng đồng loạt truy cập, kết nối TCP/IP và quá trình phân giải DNS đã thành công 100%. Tuy nhiên, khi HTTP Request tới Web Server, hệ thống Backend hoặc cơ sở dữ liệu (Database Connection Pool) bị quá tải/sập nguồn. Server gặp ngoại lệ nội bộ (Unhandled Exception) nên không thể lấy dữ liệu để phản hồi `200 OK`, mà buộc phải trả về mã **`500 Internal Server Error`**.

* **Bảng so sánh mã 500 và 404 để thông tin đến người dân:**

| Mã HTTP Status | Phân loại | Bản chất kỹ thuật | Giải thích dễ hiểu cho truyền thông |
| :--- | :--- | :--- | :--- |
| **`500 Internal Server Error`** | Server Error | Trình duyệt đã kết nối thành công đến máy chủ nhưng Server bị sự cố nội bộ/quá tải nên chưa xử lý được. | Hệ thống đang bị quá tải do số lượng truy cập cùng lúc quá lớn. Dữ liệu và hệ thống vẫn an toàn, hoàn toàn **không bị hack**. |
| **`404 Not Found`** | Client Error | Đã kết nối đến Server nhưng đường dẫn (URL) hoặc tài nguyên yêu cầu không tồn tại trên hệ thống. | Người dân nhập sai địa chỉ trang web hoặc đường dẫn đăng ký tiêm đã bị thay đổi/gỡ bỏ. |

---

### 2. Hướng dẫn sử dụng Tab Network (DevTools) chẩn đoán điểm nghẽn API

1. **Thao tác lọc:** Mở DevTools (`F12`) $\rightarrow$ Chọn tab **Network** $\rightarrow$ Click vào bộ lọc **`Fetch/XHR`** để chỉ hiển thị các truy vấn dữ liệu API đăng ký tiêm.
2. **Phân tích 2 chỉ số quan trọng trên biểu đồ Waterfall:**
   * **TTFB (Time to First Byte):** Thời gian tính từ khi trình duyệt gửi Request cho đến khi nhận được byte dữ liệu đầu tiên từ Server. Nếu TTFB kéo dài (vài giây đến vài chục giây), điểm nghẽn nằm ở **Phía Máy chủ / Database** (Server xử lý quá chậm hoặc câu truy vấn bị nghẽn).
   * **Content Download:** Thời gian trình duyệt tải toàn bộ nội dung phản hồi về máy. Nếu chỉ số này kéo dài trong khi dung lượng gói JSON nhẹ, điểm nghẽn nằm ở **Băng thông mạng** (đường truyền internet tại điểm tiêm/người dùng bị bóp nghẽn).

---

## 🔄 PHẦN 2: CHUẨN HÓA TRÌNH DUYỆT MÁY TRẠM ĐIỂM TIÊM

### 3. Khắc phục văng phiên bằng Browser Profile độc lập (`CarePulse_Station`)

* **Ý nghĩa:** Việc mở chung Profile cá nhân khiến Cookie và Session Token của hệ thống tiêm chủng bị ghi đè hoặc can thiệp bởi tiện ích mở rộng cá nhân, dẫn đến văng đăng xuất.
* **Quy trình 4 bước thiết lập Profile chuyên biệt:**
  1. Click vào biểu tượng Avatar ở góc trên bên phải trình duyệt Chrome/Edge $\rightarrow$ Chọn **Add** (Thêm).
  2. Chọn **Continue without an account** (Tiếp tục mà không cần tài khoản).
  3. Đặt tên Profile là **`CarePulse_Station`** và chọn màu sắc nhận diện riêng để tránh nhầm lẫn.
  4. Tích chọn ô **Create Desktop Shortcut** (Tạo lối tắt trên màn hình) $\rightarrow$ Nhấn **Done**. Điều dưỡng viên chỉ cần nhấp đúp vào icon này trên Desktop để truy cập thẳng môi trường làm việc sạch, không bao giờ lo bị văng phiên.

---

### 4. Giải phóng RAM và cấu hình bảo mật Phân quyền Site Access

* **Giải phóng bộ nhớ RAM (`chrome://extensions`):**
  * Nhân viên y tế truy cập `chrome://extensions` trên thanh địa chỉ.
  * Gạt công tắc **Tắt (Disable)** hoặc xóa toàn bộ các Extensions giải trí ngầm (xem phim, adblock cá nhân,...), giải phóng lập tức hàng trăm MB RAM giúp máy trạm mượt mà.
* **Cơ chế phân quyền Site Access (`On click`):**
  * **Cấu hình:** Nhấp phải vào các Extension bắt buộc giữ lại $\rightarrow$ Chọn *This can read and change site data* $\rightarrow$ Chọn **When you click the extension**.
  * **Tác dụng bảo mật:** Ngăn không cho các tiện ích tự động đọc hoặc thu thập dữ liệu ngầm từ cây DOM/Cookie trên trang `tiemchung.carepulse.vn`, bảo vệ tuyệt đối danh sách tiêm chủng cá nhân của bệnh nhân khỏi bị rò rỉ.

---

## 💡 PHẦN 3: KHẮC PHỤC LỖI BỘ NHỚ ĐỆM & VÁ GIAO DIỆN REALTIME

### 5. Phương pháp cưỡng chế xóa đệm trang web (Hard Reload)

* **Hướng dẫn phụ huynh:** Khi bảng phân bổ điểm tiêm mới không tự cập nhật do trình duyệt nạp từ Browser Cache cũ, phụ huynh thực hiện thao tác **Hard Reload**.
* **Thao tác phím tắt:** Bấm **`Ctrl + Shift + R`** (hoặc **`Ctrl + F5`**; trên macOS: **`Cmd + Shift + R`**).
* **Bản chất kỹ thuật:** Thao tác này ép trình duyệt bỏ qua toàn bộ tài nguyên tĩnh lưu trữ cục bộ (Disk Cache / Memory Cache) và gửi HTTP Request trực tiếp lên máy chủ để tải về danh sách điểm tiêm chủng cập nhật mới nhất lúc 07:30.

---

### 6. Vá lỗi giao diện nút "Xác Nhận Đăng Ký Tiêm" bằng Inspect Element & Box Model

1. **Kiểm tra thuộc tính Box Model (Tab Computed):**
   * Bấm **`Ctrl + Shift + C`** và click trực tiếp vào nút "Xác Nhận Đăng Ký Tiêm".
   * Kiểm tra sơ đồ **Box Model**: Kỹ sư phát hiện lớp **Margin** phía trên bị đặt giá trị âm (`margin-top: -50px`) hoặc lớp **Padding** quá lớn làm nút bấm bị đè bên dưới bảng danh sách điểm tiêm, khiến sự kiện click chuột không thể tác động tới nút.

2. **Chỉnh sửa CSS tạm thời (Tab Styles):**
   * Chuyển sang tab **Styles**, sửa trực tiếp đoạn mã CSS của nút bấm:
     ```css
     /* Điều chỉnh lề ngoài và lớp hiển thị */
     margin-top: 15px !important;
     padding: 10px 20px !important;
     position: relative !important;
     z-index: 9999 !important; /* Đẩy nút nổi lên trên cùng */
     ```
   * Sau khi chỉnh sửa tạm thời trên DevTools, nút bấm ngay lập tức nổi rõ trên giao diện và bấm gửi đơn thành công. Kỹ sư tiến hành cập nhật đoạn mã CSS này lên máy chủ chính thức để hoàn tất vá lỗi.
