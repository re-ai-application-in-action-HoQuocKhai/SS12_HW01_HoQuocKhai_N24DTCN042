# BÀI TẬP 01: Kết Quả Phân Tích Yêu Cầu Nghiệp Vụ Mở Tài Khoản eKYC

**Họ và tên:** Hồ Quốc Khải
**Mã sinh viên:** N24DTCN042
**Công cụ sử dụng:** Antigravity (AI Agent)
**Vai trò AI:** Senior Business Analyst – 10 năm kinh nghiệm FinTech/Digital Banking

---

## PHẦN 1: PHÂN TÍCH HỆ THỐNG eKYC

### 1.1. Actors (Các Tác Nhân Tham Gia Hệ Thống)

| # | Actor | Loại | Mô tả |
|---|-------|------|--------|
| 1 | **Khách hàng (Customer)** | Con người (Primary) | Người dùng cuối, tải app và thực hiện quy trình eKYC để mở tài khoản ngân hàng trực tuyến. |
| 2 | **Hệ thống eKYC (eKYC System)** | Hệ thống nội bộ | Hệ thống lõi xử lý luồng đăng ký, tiếp nhận dữ liệu, điều phối các bước xác thực. |
| 3 | **Hệ thống OCR (OCR Engine)** | Hệ thống con / Bên thứ 3 | Trích xuất thông tin từ ảnh chụp CCCD (họ tên, ngày sinh, số CCCD, ngày cấp, giới tính...). |
| 4 | **Hệ thống Liveness Detection** | Hệ thống con / Bên thứ 3 | Xác thực khuôn mặt thật (Anti-spoofing), đảm bảo người dùng đang thực sự có mặt trước camera. |
| 5 | **Hệ thống Face Matching** | Hệ thống con | So khớp khuôn mặt chụp trực tiếp (Selfie) với ảnh trên CCCD. |
| 6 | **CSDL Quốc gia về Dân cư (National Population DB)** | Hệ thống bên ngoài (Chính phủ) | Cơ sở dữ liệu do Bộ Công an quản lý, dùng để đối chiếu và xác minh thông tin CCCD. |
| 7 | **Hệ thống Core Banking (CBS)** | Hệ thống nội bộ | Hệ thống ngân hàng lõi, chịu trách nhiệm cấp số tài khoản, quản lý thông tin khách hàng (CIF). |
| 8 | **Quản trị viên hệ thống (System Admin)** | Con người (Secondary) | Quản lý cấu hình hệ thống eKYC, giám sát log, xử lý các trường hợp ngoại lệ bị reject tự động. |
| 9 | **Hệ thống Notification (SMS/Email Gateway)** | Hệ thống con | Gửi OTP, thông báo kết quả đăng ký, xác nhận kích hoạt tài khoản cho khách hàng. |

---

### 1.2. Business Flow (Luồng Nghiệp Vụ Chi Tiết)

#### A. Happy Path (Luồng Chính – Thành Công)

```text
[Bước 1] Khách hàng tải & mở app ABC Bank
    |
    v
[Bước 2] Khách hàng chọn "Mở tài khoản mới"
    |
    v
[Bước 3] Nhập thông tin cơ bản: Họ tên, Số điện thoại, Email
    |
    v
[Bước 4] Hệ thống gửi mã OTP qua SMS → Khách hàng nhập OTP xác thực
    |
    v
[Bước 5] Khách hàng chụp ảnh mặt trước CCCD
    |
    v
[Bước 6] Khách hàng chụp ảnh mặt sau CCCD
    |
    v
[Bước 7] Hệ thống OCR trích xuất dữ liệu từ 2 ảnh CCCD
    |
    v
[Bước 8] Hệ thống hiển thị thông tin trích xuất → Khách hàng xác nhận hoặc chỉnh sửa
    |
    v
[Bước 9] Hệ thống gọi API CSDL Quốc gia để xác minh thông tin CCCD
    |
    v
[Bước 10] Khách hàng thực hiện quét khuôn mặt (Liveness Check: chớp mắt, xoay đầu...)
    |
    v
[Bước 11] Hệ thống Face Matching: So khớp khuôn mặt Selfie với ảnh CCCD
    |
    v
[Bước 12] Hệ thống tổng hợp kết quả → Đạt yêu cầu eKYC
    |
    v
[Bước 13] Hệ thống Core Banking tự động cấp số tài khoản mới
    |
    v
[Bước 14] Gửi thông báo kết quả (SMS + Email + Push Notification)
    |
    v
[Bước 15] Khách hàng đăng nhập và sử dụng tài khoản (Trạng thái: ACTIVE)
```

#### B. Exception Path (Luồng Ngoại Lệ)

| # | Bước gốc | Ngoại lệ | Xử lý |
|---|----------|----------|-------|
| E1 | Bước 4 | OTP nhập sai 3 lần | Khóa tạm 15 phút, yêu cầu gửi lại OTP sau thời gian chờ. |
| E2 | Bước 5-6 | Ảnh CCCD bị mờ/chói/cắt góc | Thông báo lỗi cụ thể, yêu cầu chụp lại (tối đa 5 lần). |
| E3 | Bước 7 | OCR không trích xuất được dữ liệu | Chuyển trạng thái hồ sơ sang "MANUAL_REVIEW" để nhân viên kiểm tra thủ công. |
| E4 | Bước 9 | Thông tin CCCD không khớp CSDL quốc gia | Từ chối đăng ký, thông báo "Thông tin không hợp lệ", ghi nhận log cảnh báo. |
| E5 | Bước 10 | Liveness Check thất bại (nghi ngờ ảnh giả/video) | Từ chối, ghi log nghi vấn gian lận (Fraud Detection). Cho phép thử lại tối đa 3 lần. |
| E6 | Bước 11 | Face Matching có tỷ lệ khớp < 80% | Từ chối đăng ký, đề xuất khách hàng ra quầy giao dịch gần nhất. |
| E7 | Bước 9 | Timeout kết nối CSDL Quốc gia | Thông báo "Hệ thống đang bận", cho phép thử lại sau 5 phút. Ghi log timeout. |
| E8 | Bước 3 | Số điện thoại/Email đã được đăng ký | Thông báo "Số điện thoại đã tồn tại trong hệ thống", đề xuất đăng nhập hoặc liên hệ hotline. |

---

### 1.3. Functional Requirements (Yêu Cầu Chức Năng)

| Mã | Yêu cầu | Mô tả chi tiết |
|----|----------|-----------------|
| **FR-001** | Đăng ký thông tin cơ bản | Hệ thống phải cho phép khách hàng nhập: Họ tên đầy đủ, Số điện thoại (10 số, đầu số VN), Email hợp lệ. |
| **FR-002** | Xác thực OTP | Hệ thống phải gửi mã OTP 6 số qua SMS, thời hạn hiệu lực 120 giây, tự động hết hạn sau đó. |
| **FR-003** | Chụp ảnh CCCD (Mặt trước) | Hệ thống phải hỗ trợ chụp hoặc upload ảnh mặt trước CCCD, tự động kiểm tra chất lượng ảnh (độ sáng, góc, độ nét). |
| **FR-004** | Chụp ảnh CCCD (Mặt sau) | Tương tự FR-003 cho mặt sau CCCD. |
| **FR-005** | OCR trích xuất CCCD | Hệ thống OCR phải trích xuất được: Số CCCD (12 chữ số), Họ tên, Ngày sinh, Giới tính, Quê quán, Nơi thường trú, Ngày cấp, Ngày hết hạn. |
| **FR-006** | Xác nhận thông tin trích xuất | Hệ thống phải hiển thị thông tin OCR đã trích xuất để khách hàng xác nhận hoặc chỉnh sửa trước khi gửi đi. |
| **FR-007** | Đối chiếu CSDL Quốc gia | Hệ thống phải gọi API CSDL Quốc gia về Dân cư để xác minh tính hợp lệ của số CCCD và thông tin liên quan. |
| **FR-008** | Xác thực khuôn mặt (Liveness Check) | Hệ thống phải yêu cầu khách hàng thực hiện hành động ngẫu nhiên (chớp mắt, xoay đầu trái/phải, mỉm cười) để chứng minh là người thật. |
| **FR-009** | So khớp khuôn mặt (Face Matching) | Hệ thống phải so sánh khuôn mặt Selfie với ảnh trên CCCD, yêu cầu tỷ lệ khớp (Confidence Score) ≥ 80%. |
| **FR-010** | Cấp số tài khoản tự động | Nếu tất cả bước xác thực thành công, hệ thống Core Banking tự động sinh số tài khoản 13 chữ số và gán CIF (Customer Information File) mới. |
| **FR-011** | Thông báo kết quả | Hệ thống phải gửi thông báo kết quả mở tài khoản qua SMS, Email và Push Notification (thành công hoặc bị từ chối). |
| **FR-012** | Kiểm tra trùng lặp | Hệ thống phải kiểm tra xem Số CCCD hoặc Số điện thoại đã tồn tại trong hệ thống chưa trước khi cho phép tiếp tục đăng ký. |
| **FR-013** | Lưu trữ hồ sơ eKYC | Hệ thống phải lưu trữ toàn bộ hồ sơ eKYC (ảnh CCCD, ảnh Selfie, kết quả OCR, kết quả Face Matching, log thao tác) trong ít nhất 10 năm theo quy định NHNN. |

---

### 1.4. Non-Functional Requirements (Yêu Cầu Phi Chức Năng)

| Mã | Loại | Yêu cầu | Chi tiết |
|----|------|---------|----------|
| **NFR-001** | Bảo mật (Security) | Mã hóa dữ liệu truyền tải | Tất cả API phải sử dụng HTTPS/TLS 1.3. Dữ liệu nhạy cảm (ảnh CCCD, Selfie) phải được mã hóa AES-256 khi lưu trữ (Encryption at Rest). |
| **NFR-002** | Bảo mật (Security) | Bảo vệ dữ liệu cá nhân | Tuân thủ Nghị định 13/2023/NĐ-CP về Bảo vệ dữ liệu cá nhân (PDPD). Khách hàng phải đồng ý với chính sách thu thập dữ liệu trước khi bắt đầu eKYC. |
| **NFR-003** | Bảo mật (Security) | Chống gian lận (Anti-fraud) | Hệ thống phải phát hiện và chặn các hành vi giả mạo: sử dụng ảnh tĩnh, video, mặt nạ 3D thay cho khuôn mặt thật. |
| **NFR-004** | Hiệu năng (Performance) | Thời gian phản hồi API | API đăng ký cơ bản: ≤ 2 giây. API OCR xử lý CCCD: ≤ 5 giây. API Face Matching: ≤ 3 giây. Toàn bộ luồng eKYC end-to-end: ≤ 5 phút. |
| **NFR-005** | Hiệu năng (Performance) | Khả năng xử lý đồng thời | Hệ thống phải xử lý được tối thiểu 1.000 yêu cầu eKYC đồng thời (Concurrent Requests) trong giờ cao điểm. |
| **NFR-006** | Tính sẵn sàng (Availability) | Uptime cam kết | Hệ thống eKYC phải đạt SLA 99.9% uptime (tối đa 8.76 giờ downtime/năm). |
| **NFR-007** | Tính sẵn sàng (Availability) | Disaster Recovery | Hệ thống phải có cơ chế DR (Disaster Recovery) với RPO ≤ 1 giờ và RTO ≤ 4 giờ. |
| **NFR-008** | Khả năng mở rộng (Scalability) | Auto-scaling | Kiến trúc phải hỗ trợ Horizontal Scaling (tự động mở rộng pod/instance khi tải tăng đột biến). |
| **NFR-009** | Tương thích (Compatibility) | Hỗ trợ đa nền tảng | Ứng dụng phải hoạt động trên iOS 14+ và Android 10+. Hỗ trợ camera trước tối thiểu 5MP. |
| **NFR-010** | Khả năng kiểm tra (Auditability) | Ghi log đầy đủ | Mọi thao tác trong luồng eKYC phải được ghi log (Audit Trail): IP, thiết bị, timestamp, kết quả từng bước, lý do từ chối (nếu có). |

---

### 1.5. Assumptions & Business Rules (Giả Định & Quy Tắc Nghiệp Vụ)

#### A. Giả Định (Assumptions)

| # | Giả định |
|---|----------|
| A1 | Khách hàng phải là công dân Việt Nam, sở hữu CCCD gắn chip còn hiệu lực. |
| A2 | Khách hàng từ đủ 18 tuổi trở lên mới được mở tài khoản thanh toán. |
| A3 | Thiết bị di động của khách hàng có camera trước hoạt động tốt (≥ 5MP) và kết nối internet ổn định. |
| A4 | API kết nối CSDL Quốc gia về Dân cư đã được ABC Bank đăng ký và cấp quyền truy cập hợp pháp. |
| A5 | Mỗi số CCCD chỉ được phép mở tối đa 1 tài khoản thanh toán tại ABC Bank. |
| A6 | Hệ thống Core Banking (CBS) của ABC Bank đã có sẵn API cấp số tài khoản tự động. |

#### B. Quy Tắc Nghiệp Vụ (Business Rules)

| Mã | Quy tắc | Mô tả |
|----|---------|-------|
| **BR-001** | Giới hạn lần thử OTP | Khách hàng chỉ được nhập sai OTP tối đa 3 lần. Sau đó khóa tạm 15 phút. |
| **BR-002** | Giới hạn chụp CCCD | Tối đa 5 lần chụp lại CCCD cho mỗi lần đăng ký. Nếu vẫn thất bại → chuyển Manual Review. |
| **BR-003** | Ngưỡng Face Matching | Tỷ lệ khớp khuôn mặt (Confidence Score) phải ≥ 80%. Từ 60-79%: chuyển Manual Review. Dưới 60%: Từ chối trực tiếp. |
| **BR-004** | Thời hạn hoàn tất eKYC | Phiên eKYC có thời hạn tối đa 30 phút. Quá thời hạn → session hết hạn, phải bắt đầu lại. |
| **BR-005** | Chống đăng ký trùng lặp | Nếu Số CCCD hoặc Số điện thoại đã tồn tại → từ chối và đề xuất liên hệ hotline 1900-xxxx. |
| **BR-006** | Độ tuổi hợp lệ | Ngày sinh trên CCCD phải cho thấy khách hàng ≥ 18 tuổi tại thời điểm đăng ký. |
| **BR-007** | CCCD còn hiệu lực | Ngày hết hạn trên CCCD phải lớn hơn ngày hiện tại. CCCD hết hạn → từ chối. |
| **BR-008** | Trạng thái tài khoản | Tài khoản mới mở có trạng thái PENDING → chuyển ACTIVE sau khi hoàn tất tất cả bước xác thực. |
| **BR-009** | Lưu trữ dữ liệu eKYC | Toàn bộ dữ liệu eKYC phải được lưu trữ tối thiểu 10 năm kể từ ngày đóng tài khoản (theo Thông tư 16/2020/TT-NHNN). |
| **BR-010** | Zero Manual Operation | Mặc định toàn bộ quy trình tự động. Chỉ chuyển sang can thiệp thủ công (Manual Review) khi hệ thống không thể ra quyết định tự động (OCR lỗi, Face Matching trong vùng xám 60-79%). |

---

## PHẦN 2: DANH SÁCH USER STORY

### Epic 1: Đăng Ký Thông Tin Cơ Bản

#### US-001: Đăng ký tài khoản bằng thông tin cá nhân
- **Story:** As a *khách hàng mới*, I want to *nhập họ tên, số điện thoại và email để đăng ký mở tài khoản*, so that *tôi có thể bắt đầu quy trình eKYC mà không cần ra quầy ngân hàng*.
- **Priority:** Must-have
- **Acceptance Criteria:**
  - **Given** khách hàng mở app và chọn "Mở tài khoản mới", **When** khách hàng nhập đầy đủ họ tên, số điện thoại (10 số, đầu số VN), email hợp lệ, **Then** hệ thống lưu thông tin và chuyển sang bước xác thực OTP.
  - **Given** khách hàng nhập số điện thoại đã tồn tại trong hệ thống, **When** nhấn "Tiếp tục", **Then** hệ thống hiển thị thông báo "Số điện thoại đã được đăng ký" và đề xuất đăng nhập.
  - **Given** khách hàng nhập email sai định dạng, **When** nhấn "Tiếp tục", **Then** hệ thống hiển thị lỗi validation inline tại trường Email.

#### US-002: Xác thực OTP qua SMS
- **Story:** As a *khách hàng*, I want to *nhận và nhập mã OTP gửi qua SMS*, so that *hệ thống xác minh tôi là chủ sở hữu thực sự của số điện thoại đã đăng ký*.
- **Priority:** Must-have
- **Acceptance Criteria:**
  - **Given** khách hàng đã nhập thông tin hợp lệ, **When** hệ thống gửi OTP, **Then** mã OTP 6 số được gửi đến SĐT đã đăng ký trong vòng 10 giây, có hiệu lực 120 giây.
  - **Given** khách hàng nhập sai OTP 3 lần liên tiếp, **When** nhập lần thứ 4, **Then** hệ thống khóa tạm tính năng gửi OTP trong 15 phút.
  - **Given** OTP đã hết hạn, **When** khách hàng nhập mã cũ, **Then** hệ thống thông báo "Mã OTP đã hết hạn" và hiển thị nút "Gửi lại mã".

---

### Epic 2: Xác Thực CCCD (OCR)

#### US-003: Chụp ảnh mặt trước CCCD
- **Story:** As a *khách hàng*, I want to *chụp ảnh mặt trước CCCD bằng camera điện thoại*, so that *hệ thống có thể trích xuất thông tin cá nhân từ CCCD của tôi*.
- **Priority:** Must-have
- **Acceptance Criteria:**
  - **Given** khách hàng đã xác thực OTP thành công, **When** camera mở và khách hàng đặt CCCD vào khung hướng dẫn, **Then** hệ thống tự động chụp khi phát hiện CCCD nằm đúng vị trí.
  - **Given** ảnh chụp bị mờ hoặc thiếu sáng, **When** hệ thống kiểm tra chất lượng ảnh, **Then** hiển thị thông báo lỗi cụ thể (VD: "Ảnh quá mờ, vui lòng chụp lại") và cho phép chụp lại (tối đa 5 lần).

#### US-004: Chụp ảnh mặt sau CCCD
- **Story:** As a *khách hàng*, I want to *chụp ảnh mặt sau CCCD*, so that *hệ thống trích xuất thêm thông tin bổ sung (ngày cấp, đặc điểm nhận dạng)*.
- **Priority:** Must-have
- **Acceptance Criteria:**
  - **Given** khách hàng đã chụp thành công mặt trước CCCD, **When** chuyển sang bước chụp mặt sau, **Then** hệ thống hiển thị khung hướng dẫn tương tự và tự động chụp khi phát hiện mặt sau CCCD.

#### US-005: Xem và xác nhận thông tin OCR
- **Story:** As a *khách hàng*, I want to *xem lại thông tin đã được trích xuất từ CCCD và chỉnh sửa nếu sai*, so that *dữ liệu được ghi nhận chính xác trước khi gửi đi xác minh*.
- **Priority:** Must-have
- **Acceptance Criteria:**
  - **Given** hệ thống OCR đã trích xuất xong, **When** hiển thị kết quả, **Then** tất cả các trường (Họ tên, CCCD, Ngày sinh, Giới tính, Quê quán, Nơi thường trú, Ngày cấp, Ngày hết hạn) phải được hiển thị và cho phép chỉnh sửa.
  - **Given** khách hàng xác nhận thông tin, **When** nhấn "Xác nhận", **Then** hệ thống gọi API CSDL Quốc gia để đối chiếu.

---

### Epic 3: Xác Thực Khuôn Mặt (Liveness Check)

#### US-006: Quét khuôn mặt xác thực danh tính
- **Story:** As a *khách hàng*, I want to *thực hiện quét khuôn mặt bằng camera trước*, so that *hệ thống xác minh tôi chính là người sở hữu CCCD đã cung cấp*.
- **Priority:** Must-have
- **Acceptance Criteria:**
  - **Given** thông tin CCCD đã được xác minh thành công với CSDL Quốc gia, **When** khách hàng bắt đầu bước Liveness Check, **Then** hệ thống yêu cầu thực hiện hành động ngẫu nhiên (chớp mắt, xoay đầu trái/phải, mỉm cười).
  - **Given** hệ thống phát hiện sử dụng ảnh tĩnh hoặc video thay khuôn mặt thật, **When** Liveness Check thất bại, **Then** từ chối và ghi log cảnh báo gian lận (Fraud Alert). Cho phép thử lại tối đa 3 lần.
  - **Given** Liveness Check thành công, **When** hệ thống so khớp khuôn mặt, **Then** Face Matching phải đạt Confidence Score ≥ 80% so với ảnh trên CCCD.

---

### Epic 4: Kích Hoạt Tài Khoản

#### US-007: Tự động cấp số tài khoản
- **Story:** As a *khách hàng*, I want to *nhận số tài khoản ngân hàng tự động sau khi hoàn tất eKYC*, so that *tôi có thể bắt đầu sử dụng dịch vụ ngân hàng ngay lập tức*.
- **Priority:** Must-have
- **Acceptance Criteria:**
  - **Given** tất cả bước xác thực eKYC đều thành công (OTP + CCCD + Liveness + Face Matching), **When** hệ thống xác nhận hồ sơ hợp lệ, **Then** Core Banking tự động cấp số tài khoản 13 chữ số và trạng thái tài khoản chuyển từ PENDING sang ACTIVE.
  - **Given** tài khoản đã được kích hoạt, **When** gửi thông báo, **Then** khách hàng nhận thông báo qua SMS + Email + Push Notification trong vòng 30 giây.

#### US-008: Nhận thông báo kết quả
- **Story:** As a *khách hàng*, I want to *nhận thông báo về kết quả đăng ký tài khoản*, so that *tôi biết tài khoản đã được duyệt hay bị từ chối và lý do*.
- **Priority:** Should-have
- **Acceptance Criteria:**
  - **Given** eKYC thành công, **When** tài khoản được kích hoạt, **Then** thông báo gồm: số tài khoản, tên chủ tài khoản, hướng dẫn sử dụng ban đầu.
  - **Given** eKYC bị từ chối, **When** hệ thống gửi thông báo, **Then** nội dung phải ghi rõ lý do từ chối và hướng dẫn tiếp theo (liên hệ hotline hoặc ra quầy).

---

### Epic 5: Quản Trị & Giám Sát (Admin)

#### US-009: Xem và xử lý hồ sơ Manual Review
- **Story:** As a *quản trị viên hệ thống*, I want to *xem danh sách các hồ sơ eKYC chuyển sang trạng thái Manual Review*, so that *tôi có thể xử lý thủ công các trường hợp hệ thống không thể quyết định tự động*.
- **Priority:** Should-have
- **Acceptance Criteria:**
  - **Given** có hồ sơ eKYC với trạng thái MANUAL_REVIEW, **When** admin mở dashboard, **Then** hiển thị danh sách hồ sơ kèm ảnh CCCD, ảnh Selfie, kết quả OCR, Confidence Score và lý do cần review.
  - **Given** admin phê duyệt hồ sơ, **When** nhấn "Approve", **Then** hệ thống Core Banking cấp tài khoản và gửi thông báo cho khách hàng.

#### US-010: Giám sát dashboard eKYC
- **Story:** As a *quản trị viên*, I want to *xem dashboard thống kê real-time về số lượng đăng ký, tỷ lệ thành công, tỷ lệ gian lận*, so that *tôi có thể giám sát hiệu suất hệ thống eKYC*.
- **Priority:** Nice-to-have
- **Acceptance Criteria:**
  - **Given** admin truy cập trang dashboard, **When** dữ liệu load, **Then** hiển thị biểu đồ: Tổng đăng ký hôm nay, Tỷ lệ thành công (%), Tỷ lệ Fraud detected (%), Thời gian trung bình hoàn tất eKYC.

---

## PHẦN 3: DANH SÁCH USE CASE

### UC-001: Đăng Ký Thông Tin Cơ Bản

| Thuộc tính | Nội dung |
|------------|----------|
| **Use Case ID** | UC-001 |
| **Use Case Name** | Đăng Ký Thông Tin Cơ Bản |
| **Actor(s)** | Khách hàng, Hệ thống eKYC, SMS Gateway |
| **Pre-conditions** | Khách hàng đã tải và mở app ABC Bank. Khách hàng chưa có tài khoản tại ABC Bank. |
| **Post-conditions** | Thông tin cơ bản được lưu, SĐT đã được xác thực qua OTP. Hồ sơ eKYC được tạo với trạng thái PENDING. |

**Main Flow:**
1. Khách hàng chọn "Mở tài khoản mới" trên giao diện chính.
2. Hệ thống hiển thị form đăng ký gồm: Họ tên, Số điện thoại, Email.
3. Khách hàng nhập đầy đủ thông tin và nhấn "Tiếp tục".
4. Hệ thống validate dữ liệu đầu vào (định dạng SĐT, email).
5. Hệ thống kiểm tra trùng lặp SĐT/Email trong CSDL.
6. Hệ thống gửi mã OTP 6 số qua SMS đến SĐT đã nhập.
7. Khách hàng nhập mã OTP.
8. Hệ thống xác thực OTP thành công → Tạo hồ sơ eKYC (trạng thái: PENDING).
9. Chuyển sang UC-002: Upload & Đọc CCCD.

**Alternative Flow:**
- **3a.** Khách hàng muốn sửa thông tin → quay lại bước 2.
- **7a.** OTP hết hạn → Khách hàng nhấn "Gửi lại OTP" → Quay lại bước 6.

**Exception Flow:**
- **4a.** Dữ liệu không hợp lệ → Hiển thị lỗi validation inline, yêu cầu sửa lại.
- **5a.** SĐT đã tồn tại → Thông báo trùng, đề xuất đăng nhập hoặc gọi hotline.
- **7b.** OTP sai 3 lần → Khóa tạm 15 phút, hiển thị bộ đếm thời gian.

---

### UC-002: Upload & Xử Lý CCCD (OCR)

| Thuộc tính | Nội dung |
|------------|----------|
| **Use Case ID** | UC-002 |
| **Use Case Name** | Upload & Xử Lý CCCD (OCR) |
| **Actor(s)** | Khách hàng, Hệ thống eKYC, OCR Engine |
| **Pre-conditions** | Khách hàng đã xác thực OTP thành công (UC-001 hoàn tất). |
| **Post-conditions** | Ảnh CCCD 2 mặt được lưu trữ mã hóa. Thông tin cá nhân được trích xuất và xác nhận bởi khách hàng. |

**Main Flow:**
1. Hệ thống hiển thị màn hình chụp CCCD mặt trước với khung hướng dẫn.
2. Khách hàng đặt CCCD vào khung và hệ thống tự động chụp.
3. Hệ thống kiểm tra chất lượng ảnh (độ nét, góc, ánh sáng).
4. Hệ thống hiển thị màn hình chụp CCCD mặt sau → Lặp lại bước 2-3.
5. OCR Engine trích xuất thông tin từ 2 ảnh.
6. Hệ thống hiển thị kết quả OCR để khách hàng xem và xác nhận.
7. Khách hàng nhấn "Xác nhận" → Hệ thống gọi API CSDL Quốc gia đối chiếu.
8. CSDL Quốc gia xác nhận khớp → Chuyển sang UC-003.

**Alternative Flow:**
- **6a.** Khách hàng phát hiện OCR sai → Chỉnh sửa thủ công trường bị sai → Quay lại bước 7.

**Exception Flow:**
- **3a.** Ảnh chất lượng kém → Thông báo lỗi cụ thể, cho phép chụp lại (tối đa 5 lần).
- **3b.** Chụp lại 5 lần vẫn thất bại → Chuyển hồ sơ sang MANUAL_REVIEW.
- **5a.** OCR không trích xuất được dữ liệu → Chuyển MANUAL_REVIEW.
- **8a.** Thông tin không khớp CSDL Quốc gia → Từ chối đăng ký, ghi log cảnh báo.
- **8b.** API CSDL Quốc gia timeout → Thông báo "Hệ thống đang bận", cho thử lại sau 5 phút.

---

### UC-003: Xác Thực Khuôn Mặt (Liveness Check & Face Matching)

| Thuộc tính | Nội dung |
|------------|----------|
| **Use Case ID** | UC-003 |
| **Use Case Name** | Xác Thực Khuôn Mặt |
| **Actor(s)** | Khách hàng, Liveness Detection System, Face Matching System |
| **Pre-conditions** | CCCD đã được xác minh thành công với CSDL Quốc gia (UC-002 hoàn tất). |
| **Post-conditions** | Khuôn mặt khách hàng được xác thực là người thật và khớp với ảnh trên CCCD. |

**Main Flow:**
1. Hệ thống mở camera trước và hiển thị khung oval hướng dẫn đặt khuôn mặt.
2. Hệ thống yêu cầu khách hàng thực hiện hành động ngẫu nhiên (VD: "Hãy chớp mắt").
3. Liveness Detection xác nhận đây là người thật (không phải ảnh/video/mặt nạ).
4. Hệ thống chụp ảnh Selfie HD.
5. Face Matching so khớp Selfie với ảnh trên CCCD → Confidence Score ≥ 80%.
6. Xác thực thành công → Chuyển sang UC-004.

**Alternative Flow:**
- **2a.** Khách hàng không thực hiện đúng hành động → Yêu cầu hành động khác (VD: "Hãy xoay đầu sang trái").

**Exception Flow:**
- **3a.** Liveness Check phát hiện ảnh giả/video → Từ chối, ghi log Fraud Alert. Cho phép thử lại tối đa 3 lần.
- **3b.** Thử lại 3 lần vẫn thất bại → Từ chối đăng ký, đề xuất ra quầy giao dịch.
- **5a.** Confidence Score 60-79% → Chuyển sang MANUAL_REVIEW.
- **5b.** Confidence Score < 60% → Từ chối trực tiếp, ghi log.

---

### UC-004: Kích Hoạt Tài Khoản

| Thuộc tính | Nội dung |
|------------|----------|
| **Use Case ID** | UC-004 |
| **Use Case Name** | Kích Hoạt Tài Khoản Tự Động |
| **Actor(s)** | Hệ thống eKYC, Core Banking System, Notification Gateway |
| **Pre-conditions** | Tất cả bước xác thực eKYC (UC-001, UC-002, UC-003) đều thành công. |
| **Post-conditions** | Tài khoản ngân hàng được cấp số, trạng thái ACTIVE. Khách hàng nhận thông báo. |

**Main Flow:**
1. Hệ thống eKYC tổng hợp kết quả toàn bộ quy trình → Hồ sơ đạt yêu cầu.
2. Hệ thống gọi API Core Banking để tạo CIF (Customer Information File) mới.
3. Core Banking tự động sinh số tài khoản 13 chữ số.
4. Trạng thái tài khoản chuyển từ PENDING → ACTIVE.
5. Hệ thống gửi thông báo qua SMS + Email + Push Notification: Số tài khoản, tên chủ tài khoản, hướng dẫn sử dụng.
6. Khách hàng đăng nhập app và bắt đầu sử dụng.

**Alternative Flow:**
- **1a.** Hồ sơ ở trạng thái MANUAL_REVIEW → Admin xem xét và quyết định Approve/Reject.

**Exception Flow:**
- **2a.** Core Banking trả lỗi (VD: hệ thống bận) → Retry tự động 3 lần, nếu vẫn thất bại → Ghi log, thông báo khách hàng "Hệ thống đang xử lý, vui lòng chờ".
- **5a.** SMS Gateway lỗi → Fallback sang Email/Push. Ghi log lỗi gửi SMS.

---

### UC-005: Xử Lý Hồ Sơ Manual Review (Admin)

| Thuộc tính | Nội dung |
|------------|----------|
| **Use Case ID** | UC-005 |
| **Use Case Name** | Xử Lý Hồ Sơ Manual Review |
| **Actor(s)** | Quản trị viên hệ thống (Admin), Hệ thống eKYC, Core Banking |
| **Pre-conditions** | Có hồ sơ eKYC ở trạng thái MANUAL_REVIEW (do OCR lỗi hoặc Face Matching nằm trong vùng xám 60-79%). |
| **Post-conditions** | Hồ sơ được phê duyệt (Approve → cấp tài khoản) hoặc bị từ chối (Reject → thông báo khách hàng). |

**Main Flow:**
1. Admin đăng nhập hệ thống quản trị.
2. Mở danh sách hồ sơ MANUAL_REVIEW (sắp xếp theo thời gian).
3. Admin xem chi tiết hồ sơ: ảnh CCCD, ảnh Selfie, thông tin OCR, Confidence Score.
4. Admin đối chiếu thủ công và ra quyết định Approve hoặc Reject.
5. Nếu Approve → Hệ thống kích hoạt UC-004 (Cấp tài khoản).
6. Nếu Reject → Hệ thống gửi thông báo từ chối cho khách hàng kèm lý do.

**Alternative Flow:**
- **4a.** Admin cần thêm thông tin → Đặt trạng thái "NEED_MORE_INFO", hệ thống gửi yêu cầu bổ sung cho khách hàng.

**Exception Flow:**
- **3a.** Ảnh CCCD/Selfie bị lỗi không mở được → Ghi log, đánh dấu hồ sơ lỗi hệ thống, yêu cầu khách hàng đăng ký lại.
