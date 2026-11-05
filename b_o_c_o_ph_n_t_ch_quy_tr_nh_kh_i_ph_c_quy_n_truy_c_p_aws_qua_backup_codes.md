# BÁO CÁO PHÂN TÍCH QUY TRÌNH XỬ LÝ SỰ CỐ: KHÔI PHỤC QUYỀN TRUY CẬP AWS KHI MẤT THIẾT BỊ 2FA

**Chức danh:** Lead Developer  
**Hệ thống:** Quản trị đám mây AWS (AWS Cloud Management)  
**Tình huống:** Mất thiết bị di động chứa ứng dụng Google Authenticator (MFA)  
**Tiêu chuẩn bảo mật:** ISO/IEC 27001 - Incident Management & AWS IAM Best Practices  

---

## 1. BÁO CÁO PHÂN TÍCH THEO MÔ HÌNH IPO (INPUT - PROCESS - OUTPUT)

```
+-------------------------------------------------------------------------+
|                                 INPUT                                   |
| - Bộ mã dự phòng (Backup Codes) lưu trữ ngoại tuyến.                     |
| - Tài khoản IAM / Root Credentials.                                     |
+-------------------------------------------------------------------------+
                                     │
                                     ▼
+-------------------------------------------------------------------------+
|                                PROCESS                                  |
| 1. Xác thực khẩn cấp qua Backup Code (One-time use).                    |
| 2. Thu hồi (Deactivate) và hủy liên kết thiết bị cũ.                    |
| 3. Cấp phát (Provision) & liên kết thiết bị 2FA mới.                    |
| 4. Sinh bộ Backup Codes mới & lưu trữ an toàn.                          |
+-------------------------------------------------------------------------+
                                     │
                                     ▼
+-------------------------------------------------------------------------+
|                                OUTPUT                                   |
| - Hệ thống AWS an toàn (Bảo vệ triệt để trước thiết bị bị thất lạc).    |
| - Khôi phục hoàn toàn quyền truy cập vận hành.                          |
| - Nhật ký kiểm toán bảo mật (Audit Log) trên CloudTrail.                 |
+-------------------------------------------------------------------------+
```

---

### 1.1. Input (Đầu vào)

1. **Bộ mã dự phòng (Backup Codes):**
   * Chuỗi mã xác thực một lần (One-Time Passcodes) được tạo sẵn khi thiết lập 2FA/MFA lần đầu.
   * Yêu cầu điều kiện: Mã chưa từng qua sử dụng và được lưu trữ ở một kênh cách ly hoàn toàn với thiết bị đã bị mất.
2. **Tài khoản định danh thay thế / Quy trình ủy quyền (Emergency Access Escalation):**
   * Trong trường hợp người dùng mất cả Backup Codes, cần Input bổ sung là xác thực từ Tài khoản Root hoặc tài khoản Administrator đại diện trong tổ chức để phê duyệt yêu cầu reset MFA.

---

### 1.2. Process (Quy trình xử lý bảo mật 4 bước)

#### Bước 1: Đăng nhập khẩn cấp bằng Backup Code (Emergency Authentication)
* Truy cập trang đăng nhập AWS Console bằng Username & Password của tài khoản.
* Tại giao diện yêu cầu mã OTP 6 chữ số, chọn tùy chọn **"Troubleshoot MFA"** hoặc **"Use Backup Code"**.
* Nhập 01 mã dự phòng hợp lệ. Ngay khi đăng nhập thành công, hệ thống AWS tự động hủy hiệu lực của riêng mã này để tránh bị khai thác lại (Replay Attack).

#### Bước 2: Thu hồi và Hủy bỏ kết nối thiết bị cũ (Revoke Compromised MFA Device)
* Ngay sau khi truy cập thành công vào AWS Management Console, truy cập dịch vụ **IAM (Identity and Access Management)** $\rightarrow$ **Users** $\rightarrow$ Chọn tài khoản $\rightarrow$ Thẻ **Security credentials**.
* Tại phần **Multi-factor authentication (MFA)**, chọn thiết bị Google Authenticator gắn liền với chiếc điện thoại đã mất và nhấn **Deactivate** (Hủy kích hoạt), sau đó chọn **Delete** (Xóa hoàn toàn).
* *Mục tiêu:* Vô hiệu hóa ngay lập tức khả năng sinh mã OTP trên điện thoại đã mất, triệt tiêu nguy cơ nếu kẻ nhặt được điện thoại bẻ khóa thành công thiết bị.

#### Bước 3: Cấp phát và Liên kết thiết bị MFA mới (Provisioning & Re-binding)
* Cài đặt ứng dụng Google Authenticator (hoặc phần mềm MFA tương đương) trên thiết bị di động mới.
* Chọn **Assign MFA device** trên AWS IAM Console.
* Nhập tên thiết bị mới, chọn loại MFA (Authenticators App) và hiển thị Mã QR (QR Code).
* Quét mã QR bằng ứng dụng trên điện thoại mới, sau đó nhập 02 mã OTP liên tiếp sinh ra từ ứng dụng để hoàn tất xác minh đồng bộ.

#### Bước 4: Tạo và Bảo đảm lưu trữ bộ Backup Codes mới (Re-generate Backup Codes)
* Thực hiện tạo lại (Generate) bộ mã dự phòng mới từ hệ thống.
* Ngay lập tức ghi chép/in ra bản cứng (Hard Copy) hoặc lưu trữ trong thiết bị mã hóa ngoại tuyến.
* Đánh dấu hủy toàn bộ danh sách Backup Codes cũ nhằm đảm bảo tính toàn vẹn bảo mật.

---

### 1.3. Output (Đầu ra)

1. **Hệ thống an toàn (System Security Preserved):** 
   * Thiết bị cũ bị cách ly hoàn toàn. Dữ liệu và hạ tầng AWS được bảo đảm an toàn trước mọi nguy cơ rò rỉ từ chiếc điện thoại bị thất lạc.
2. **Khôi phục quyền truy cập (Access Restored):**
   * Lead Developer lấy lại đầy đủ quyền vận hành và bảo trì hệ thống mà không làm gián đoạn tiến độ dự án.
3. **Nhật ký kiểm toán an ninh (Security Audit Trail):**
   * AWS CloudTrail ghi nhận chính xác thời gian, IP và hành động hủy/tạo mới MFA, phục vụ mục đích rà soát an ninh định kỳ.

---

## 2. PHÂN TÍCH RỦI RÀO BẢO MẬT: TẠI SAO KHÔNG ĐƯỢC LƯU BACKUP CODES DƯỚI DẠNG ẢNH TRONG THƯ VIỆN ĐIỆN THOẠI?

Việc lưu trữ Backup Codes dưới dạng ảnh chụp màn hình (Screenshot) hoặc tệp hình ảnh trong Thư viện (Photos Gallery) trên điện thoại vi phạm nghiêm trọng nguyên tắc **Phân tách rủi ro (Risk Separation)** trong an toàn thông tin với 3 nguyên nhân cốt lõi sau:

### 2.1. Triệt tiêu bản chất của Xác thực 2 Yếu tố (Destruction of 2FA Concept)
Xác thực hai yếu tố (2FA) dựa trên 2 yếu tố độc lập:
* **Cái bạn biết (What you know):** Mật khẩu tài khoản (Password).
* **Cái bạn có (What you have):** Thiết bị sinh mã OTP / Backup Codes.

Nếu bạn lưu ảnh Backup Codes ngay trên điện thoại chứa ứng dụng Google Authenticator:
* Khi kẻ gian có được điện thoại (qua trộm cắp/thất lạc) và bẻ khóa được thiết bị, họ có **toàn bộ các yếu tố xác thực** cùng một lúc.
* Mất điện thoại đồng nghĩa với việc **mất hoàn toàn khả năng bảo vệ độc lập** của tài khoản.

### 2.2. Nguy cơ rò rỉ tự động qua Tự động Đồng bộ Đám mây (Cloud Auto-Sync Vulnerability)
Các hệ điều hành di động hiện đại (iOS, Android) đều bật mặc định tính năng tự động sao lưu hình ảnh lên mây (iCloud Photos, Google Photos, OneDrive...).
* **Kịch bản tấn công:** Kẻ tấn công không cần lấy chiếc điện thoại vật lý của bạn. Họ chỉ cần chiếm đoạt hoặc brute-force tài khoản iCloud/Google của bạn là đã có thể tải về toàn bộ ảnh chụp màn hình chứa Backup Codes.
* **Hậu quả:** Kẻ tấn công có được chìa khóa vào AWS từ xa mà bạn không hề hay biết.

### 2.3. Mã độc và Quyền truy cập của Ứng dụng bên thứ ba (Malware & App Permissions Risk)
Rất nhiều ứng dụng di động (chỉnh sửa ảnh, trò chơi, công cụ tiện ích) yêu cầu cấp quyền `READ_EXTERNAL_STORAGE` hoặc truy cập toàn bộ Thư viện ảnh.
* **Nguy cơ:** Các ứng dụng chứa mã độc (Spyware/Trojan) hoặc các ứng dụng bị khai thác lỗ hổng có thể âm thầm quét (scan) các hình ảnh trong thư viện để tìm kiếm chuỗi ký tự dạng Backup Codes / QR Codes và gửi về máy chủ của kẻ tấn công (C&C Server).

---

## 3. KHUYẾN NGHỊ PHƯƠNG PHÁP LƯU TRỮ BACKUP CODES CHUẨN DOANH NGHIỆP

| Phương pháp | Mức độ an toàn | Mô tả triển khai |
| :--- | :---: | :--- |
| **Bản in giấy (Physical Hardcopy)** | **Rất cao (Highly Recommended)** | In bộ mã ra giấy, cất giữ trong két sắt hoặc tủ tài liệu có khóa an toàn tại văn phòng/nhà riêng. |
| **Trình quản lý mật khẩu (Password Manager)** | **Cao** | Sử dụng các giải pháp chuyên dụng doanh nghiệp (1Password, Bitwarden) có tính năng mã hóa End-to-End và bảo vệ bằng Master Password + Master Key độc lập. |
| **Thiết bị lưu trữ mã hóa ngoại tuyến** | **Cao** | Lưu tệp mã hóa trong USB có bảo mật phần cứng (Hardware Encrypted Drive) không kết nối Internet thường xuyên. |