# Software Requirements Specification (SRS)
## Module: Login (Đăng nhập)
### Hệ thống: Perfex CRM – Anh Tester Demo
**URL khảo sát:** https://crm.anhtester.com/authentication/login

---

## 1. Giới thiệu

### 1.1 Mục đích
Tài liệu này mô tả các yêu cầu chức năng và phi chức năng của module **Đăng nhập (Login)** trên hệ thống CRM demo (Perfex CRM – Anh Tester Demo), phục vụ cho việc thiết kế test case, kiểm thử tự động và kiểm thử thủ công.

### 1.2 Phạm vi
Tài liệu bao gồm:
- Màn hình **Đăng nhập** (`/authentication/login`)
- Luồng liên quan: **Quên mật khẩu** (`/authentication/forgot_password`)

Không bao gồm: màn hình sau khi đăng nhập thành công (Dashboard), chức năng đăng ký tài khoản mới (hệ thống không hỗ trợ self sign-up cho CRM nội bộ).

### 1.3 Đối tượng sử dụng
- Nhân viên/Admin nội bộ sử dụng CRM (staff).
- (Ngoài phạm vi khảo sát: khách hàng đăng nhập Customer Portal, nếu có domain/route riêng.)

### 1.4 Ghi chú khảo sát & giới hạn
- Việc khảo sát được thực hiện bằng cách truy cập trực tiếp trang và quan sát hành vi UI (không có quyền truy cập mã nguồn).
- Tài khoản demo được cung cấp để thử nghiệm (`admin@example.com` / `123456`) **đăng nhập không thành công** tại thời điểm khảo sát (hệ thống trả về thông báo *"Invalid username or password"*). Đây là site demo công khai nên khả năng cao tài khoản đã bị người dùng khác đổi mật khẩu. Do đó, màn hình **sau khi đăng nhập thành công** (Dashboard, luồng đăng xuất, khóa tài khoản sau N lần sai...) **chưa được xác minh trực tiếp** và được mô tả dựa trên hành vi chuẩn của Perfex CRM — cần được tester xác nhận lại khi có tài khoản hợp lệ.

---

## 2. Mô tả tổng quan

### 2.1 Bối cảnh sản phẩm
Màn hình Login là điểm vào (entry point) của hệ thống CRM, xác thực người dùng bằng **Email + Mật khẩu** trước khi cấp quyền truy cập các module nghiệp vụ khác (Khách hàng, Hợp đồng, Hóa đơn...).

### 2.2 Các thành phần giao diện quan sát được

| # | Thành phần | Loại control | Bắt buộc | Ghi chú |
|---|---|---|---|---|
| 1 | Logo / link về trang chủ | Link | - | Điều hướng về `/` |
| 2 | Knowledge Base | Link | - | Điều hướng tới trang kiến thức |
| 3 | Language | Dropdown (select) | Không | Mặc định **English**; danh sách hỗ trợ 26 ngôn ngữ (English, Chinese, Vietnamese, Indonesia, Bulgarian, Portuguese_br, Swedish, Dutch, Persian, Turkish, Catalan, Spanish, Portuguese, French, Czech, Japanese, Italian, Slovak, Russian, Polish, Greek, Finnish, Norwegian, German, Ukrainian, Romanian) |
| 4 | Email Address | Textbox (text) | Có | Không có ràng buộc định dạng email rõ ràng ở phía client (input type="text", không phải type="email") |
| 5 | Password | Textbox (password, ẩn ký tự) | Có | |
| 6 | Remember me | Checkbox | Không | |
| 7 | Nút **Login** | Submit button | - | |
| 8 | Link **Forgot Password?** | Link | - | Điều hướng tới `/authentication/forgot_password` |
| 9 | Footer copyright | Text | - | |

### 2.3 Ràng buộc
- Hệ thống dùng CSRF token ẩn (hidden input) trong form — mỗi lần tải lại trang sẽ có token mới.
- Giao diện responsive cơ bản (có nút "Toggle navigation" cho màn hình nhỏ).

---

## 3. Yêu cầu chức năng

### 3.1 FR-01: Đăng nhập với thông tin hợp lệ
**Mô tả:** Người dùng nhập đúng Email và Mật khẩu đã đăng ký, bấm **Login**.
**Kết quả mong đợi:** Hệ thống xác thực thành công, chuyển hướng người dùng vào trang Dashboard/trang chủ sau đăng nhập, tạo phiên làm việc (session).
**Trạng thái khảo sát:** ⚠️ Chưa xác minh được do tài khoản demo cung cấp không hoạt động (xem mục 1.4).

### 3.2 FR-02: Validate khi bỏ trống trường bắt buộc
**Mô tả:** Người dùng bấm **Login** khi để trống Email và/hoặc Password.
**Kết quả quan sát được:**
- Trường Email trống → hiển thị thông báo inline: **"The Email Address field is required."**
- Trường Password trống → hiển thị thông báo inline: **"The Password field is required."**
- Validate được thực hiện ngay khi submit (client-side), không reload trang, không mất dữ liệu đã nhập ở trường còn lại.

### 3.3 FR-03: Đăng nhập với thông tin sai
**Mô tả:** Người dùng nhập Email và/hoặc Password không đúng với tài khoản tồn tại, bấm **Login**.
**Kết quả quan sát được:**
- Trang submit (reload) và hiển thị **thông báo dạng toast/alert phía trên form**: **"Invalid username or password"**, kèm nút **Close (×)** để đóng thông báo.
- Các trường Email/Password trên form bị **xóa trắng** sau khi submit thất bại (không giữ lại giá trị đã nhập — người dùng phải nhập lại từ đầu).
- Hệ thống **không phân biệt** rõ giữa "sai email" và "sai mật khẩu" (cùng một thông báo chung) — đây là hành vi bảo mật hợp lý (tránh lộ thông tin tài khoản nào tồn tại).

### 3.4 FR-04: Ghi nhớ đăng nhập (Remember me)
**Mô tả:** Người dùng tick chọn **Remember me** trước khi đăng nhập.
**Kết quả mong đợi:** Phiên đăng nhập được lưu lâu hơn (cookie/token có thời hạn dài hơn so với không tick).
**Trạng thái khảo sát:** ⚠️ Chưa xác minh được (phụ thuộc FR-01).

### 3.5 FR-05: Đổi ngôn ngữ hiển thị
**Mô tả:** Người dùng chọn ngôn ngữ khác trong dropdown **Language** trước khi đăng nhập.
**Kết quả mong đợi:** Giao diện (nhãn, placeholder, thông báo lỗi) đổi sang ngôn ngữ tương ứng.
**Trạng thái khảo sát:** ⚠️ Chỉ xác minh được danh sách lựa chọn, chưa xác minh nội dung dịch cho từng ngôn ngữ.

### 3.6 FR-06: Quên mật khẩu (liên quan)
**Mô tả:** Người dùng bấm link **Forgot Password?**, được điều hướng tới trang `/authentication/forgot_password` gồm:
- 1 trường **Email Address** (input type="email" — có ràng buộc định dạng email ở mức HTML5, khác với trang Login).
- 1 nút **Submit**.
**Kết quả mong đợi:** Hệ thống gửi email chứa link đặt lại mật khẩu tới địa chỉ email nếu tồn tại trong hệ thống.
**Trạng thái khảo sát:** ⚠️ Chưa submit thực tế để tránh gửi email ngoài ý muốn trên hệ thống demo công khai; cần kiểm thử riêng trong môi trường test có kiểm soát.

### 3.7 FR-07: Điều hướng khi đã có trạng thái Login ở header
**Mô tả:** Trang chủ/Knowledge Base hiển thị nút/link **Login** ở góc phải thanh điều hướng khi chưa đăng nhập.
**Kết quả mong đợi:** Sau khi đăng nhập thành công, khu vực này đổi thành avatar/tên người dùng hoặc menu tài khoản.
**Trạng thái khảo sát:** ⚠️ Chưa xác minh (phụ thuộc FR-01).

---

## 4. Yêu cầu phi chức năng

| Hạng mục | Yêu cầu |
|---|---|
| Bảo mật | Password được ẩn ký tự khi nhập; có CSRF token trong form; thông báo lỗi đăng nhập không tiết lộ email có tồn tại hay không |
| Bảo mật (khuyến nghị) | Nên có giới hạn số lần đăng nhập sai liên tiếp (rate limiting / account lockout / CAPTCHA) để chống brute-force — **cần kiểm thử riêng, không nên thử nhiều lần trên site demo công khai vì ảnh hưởng người dùng khác** |
| Hiệu năng | Trang Login phải phản hồi (hiển thị lỗi hoặc chuyển trang) trong thời gian hợp lý (khuyến nghị < 3s) |
| Khả dụng (Usability) | Thông báo lỗi rõ ràng, hiển thị gần trường liên quan (lỗi required) hoặc nổi bật đầu trang (lỗi sai thông tin) |
| Đa ngôn ngữ | Hỗ trợ tối thiểu 26 ngôn ngữ ở mức lựa chọn giao diện |
| Tương thích | Giao diện có responsive toggle cho thiết bị di động/màn hình nhỏ |

---

## 5. Quy tắc validate (Validation Rules)

| Trường | Bắt buộc | Định dạng | Thông báo lỗi quan sát được |
|---|---|---|---|
| Email Address (Login) | Có | Không ràng buộc định dạng email ở client (input type="text") | "The Email Address field is required." |
| Password (Login) | Có | Không giới hạn độ dài tối thiểu/tối đa quan sát được qua UI | "The Password field is required." |
| Email, Password sai | - | - | "Invalid username or password" (toast, có nút Close) |
| Email Address (Forgot Password) | Có (suy luận) | input type="email" → có validate định dạng email cơ bản của trình duyệt | Chưa xác minh (chưa submit) |

---

## 6. Use Case chính: Đăng nhập hệ thống

**Tên Use Case:** UC-01 Đăng nhập
**Tác nhân:** Người dùng CRM (Staff)
**Điều kiện tiên quyết:** Người dùng đã có tài khoản hợp lệ, truy cập được `/authentication/login`

**Luồng chính (Main flow):**
1. Người dùng truy cập trang Login.
2. Hệ thống hiển thị form gồm Language, Email Address, Password, Remember me, nút Login, link Forgot Password.
3. Người dùng nhập Email và Password hợp lệ.
4. Người dùng bấm **Login**.
5. Hệ thống xác thực thông tin.
6. Hệ thống chuyển hướng người dùng vào trang chính sau đăng nhập.

**Luồng phụ (Alternative flows):**
- **A1 – Bỏ trống trường bắt buộc:** Ở bước 3, người dùng để trống Email/Password → hệ thống hiển thị lỗi required tương ứng (xem FR-02), dừng tại form Login.
- **A2 – Sai thông tin đăng nhập:** Ở bước 5, thông tin không khớp → hệ thống hiển thị "Invalid username or password", xóa trắng form, dừng tại trang Login (xem FR-03).
- **A3 – Quên mật khẩu:** Người dùng bấm "Forgot Password?" thay vì nhập thông tin → chuyển sang luồng UC-02 (xem FR-06).

**Điều kiện kết thúc:** Người dùng ở trang chính của hệ thống (thành công) hoặc vẫn ở trang Login kèm thông báo lỗi (thất bại).

---

## 7. Danh sách câu hỏi / điểm cần làm rõ (Open Questions)

1. Hệ thống có giới hạn số lần đăng nhập sai trước khi khóa tài khoản tạm thời không? Thông báo khi bị khóa là gì?
2. Mật khẩu có yêu cầu độ phức tạp tối thiểu khi tạo mới không? (Không thể kiểm tra qua màn hình Login)
3. Trường Email ở Login có phân biệt hoa/thường không?
4. Sau khi tick "Remember me", thời gian duy trì phiên đăng nhập là bao lâu?
5. Có hỗ trợ đăng nhập bằng tài khoản mạng xã hội (Google/Facebook SSO) không? (Không quan sát thấy trên UI hiện tại)
6. Trang Customer Portal (nếu có) có dùng chung màn hình Login này hay route riêng?

---

## 8. Phụ lục

### 8.1 Dữ liệu kiểm thử đã dùng khi khảo sát
| Trường hợp | Email | Password | Kết quả |
|---|---|---|---|
| Để trống | (trống) | (trống) | Lỗi required trên cả 2 trường |
| Sai thông tin | wrong@example.com | wrongpass123 | "Invalid username or password" |
| Tài khoản demo được cung cấp | admin@example.com | 123456 | "Invalid username or password" (không đăng nhập được tại thời điểm khảo sát) |

### 8.2 Môi trường khảo sát
- URL: https://crm.anhtester.com/authentication/login
- Thời điểm: 2026-11-21
- Công cụ: trình duyệt tự động (Claude Browser), độ phân giải viewport 1024×768
