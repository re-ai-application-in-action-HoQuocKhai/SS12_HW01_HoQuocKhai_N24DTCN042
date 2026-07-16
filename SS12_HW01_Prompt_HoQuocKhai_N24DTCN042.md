# BÀI TẬP 01 - FILE PROMPT: Phân Tích Yêu Cầu Nghiệp Vụ Mở Tài Khoản eKYC

**Họ và tên:** Hồ Quốc Khải
**Mã sinh viên:** N24DTCN042

---

## Prompt 1: Thiết lập vai trò & Phân tích hệ thống toàn diện

```text
Hãy đóng vai một Senior Business Analyst có 10 năm kinh nghiệm trong lĩnh vực FinTech và Ngân hàng số (Digital Banking).

[Ngữ cảnh]:
Ngân hàng số ABC Bank đang muốn số hóa hoàn toàn quy trình mở thẻ ngân hàng. Thay vì khách hàng phải ra quầy giao dịch, họ có thể mở tài khoản thông qua quy trình eKYC (Electronic Know Your Customer) trên ứng dụng di động. Luồng cơ bản như sau:
1. Khách hàng tải app ABC Bank.
2. Đăng ký thông tin cơ bản (họ tên, email, số điện thoại).
3. Chụp ảnh 2 mặt CCCD (Căn cước công dân).
4. Quét khuôn mặt để xác thực danh tính (Liveness Check).
5. Hệ thống đối chiếu dữ liệu CCCD với khuôn mặt.
6. Cấp số tài khoản tự động nếu hợp lệ.
Mục tiêu cốt lõi của ngân hàng là "Zero Manual Operation" – giảm thiểu tối đa sự can thiệp của giao dịch viên (Teller).

[Mục tiêu]:
Phân tích toàn diện hệ thống eKYC này và trả về đầy đủ các thành phần sau:
1. **Actors (Các tác nhân):** Liệt kê và mô tả tất cả các tác nhân tham gia hệ thống (cả con người và hệ thống bên ngoài).
2. **Business Flow (Luồng nghiệp vụ):** Mô tả luồng nghiệp vụ từng bước chi tiết (Happy Path và Exception Path).
3. **Functional Requirements (Yêu cầu chức năng):** Liệt kê các yêu cầu chức năng, đánh mã FR-001, FR-002...
4. **Non-Functional Requirements (Yêu cầu phi chức năng):** Liệt kê các yêu cầu về bảo mật, hiệu năng, tính sẵn sàng, khả năng mở rộng. Đánh mã NFR-001, NFR-002...
5. **Assumptions & Business Rules (Giả định và Quy tắc nghiệp vụ):** Liệt kê các giả định về môi trường, luật pháp và các quy tắc nghiệp vụ của ngân hàng.

[Ràng buộc]:
- Phải tuân thủ quy định của Ngân hàng Nhà nước Việt Nam (NHNN) về eKYC.
- Phải đề cập đến tích hợp với hệ thống xác thực CCCD quốc gia (CSDL quốc gia về dân cư).
- Phải cân nhắc về Luật An ninh mạng và bảo vệ dữ liệu cá nhân (PDPD) của Việt Nam.

[Định dạng]: Trình bày dạng Markdown có heading rõ ràng, sử dụng bảng khi cần thiết, đánh số thứ tự cho từng mục.
```

---

## Prompt 2: Chuyển đổi sang User Story

```text
Dựa trên kết quả phân tích nghiệp vụ eKYC ở trên, hãy tiếp tục đóng vai Senior BA và chuyển đổi các Functional Requirements thành danh sách User Story theo chuẩn quốc tế:

[Định dạng User Story]:
"As a [vai trò], I want to [hành động], so that [mục đích/giá trị]."

[Ràng buộc]:
- Mỗi User Story phải kèm theo Acceptance Criteria (Tiêu chí chấp nhận) dạng Given-When-Then.
- Phải đánh mã ID cho từng Story (US-001, US-002...).
- Phân nhóm các User Story theo Epic (nhóm tính năng lớn): Đăng ký, Xác thực CCCD, Xác thực khuôn mặt, Kích hoạt tài khoản.
- Gắn mức độ ưu tiên (Priority): Must-have, Should-have, Nice-to-have.

[Định dạng]: Trình bày dạng Markdown, mỗi User Story là một khối riêng biệt có ID, nội dung, Acceptance Criteria và Priority.
```

---

## Prompt 3: Chuyển đổi sang Use Case

```text
Tiếp tục đóng vai Senior BA. Từ phân tích nghiệp vụ và danh sách User Story đã có, hãy sinh ra danh sách Use Case chi tiết cho hệ thống eKYC.

[Định dạng mỗi Use Case]:
- **Use Case ID:** UC-0XX
- **Use Case Name:** Tên ngắn gọn
- **Actor(s):** Tác nhân tham gia
- **Pre-conditions:** Điều kiện tiên quyết
- **Post-conditions:** Kết quả sau khi hoàn thành
- **Main Flow (Luồng chính):** Các bước tuần tự đánh số 1, 2, 3...
- **Alternative Flow (Luồng thay thế):** Các nhánh rẽ hợp lệ
- **Exception Flow (Luồng ngoại lệ):** Các trường hợp lỗi/thất bại

[Ràng buộc]:
- Phải bao phủ tối thiểu 5 Use Case chính: Đăng ký thông tin, Upload CCCD, OCR xử lý CCCD, Xác thực khuôn mặt (Liveness), Kích hoạt tài khoản.
- Mỗi Use Case phải có ít nhất 1 Alternative Flow và 1 Exception Flow.

[Định dạng]: Markdown với heading và danh sách có thứ tự rõ ràng.
```
