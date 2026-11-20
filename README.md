# Đề 4 · Chuyển khoản QR

> **Mã:** DE04 · **Độ khó:** 4/8 · **Vai trò:** 1 (người chuyển tiền) · **Nền tảng:** App ngân hàng iOS / Android

Tính năng chuyển khoản bằng mã QR trong app ngân hàng, theo đúng các bước người dùng làm: **quét mã QR → chọn số tiền → xác thực và xác nhận chuyển khoản → hoàn thành**. Đây là đề đầu tiên có **tiền thật đi qua**, xác thực nhiều mức theo quy định và trạng thái "chưa xác định" khi hệ thống chưa có kết quả. Đề gồm 4 tính năng, mỗi bước là một tính năng; mỗi tính năng có 1 PRD và 1 SRS riêng.

## Tính năng

| **Mã** | **Bước** | **Phạm vi** | **PRD** | **SRS** |
| --- | --- | --- | --- | --- |
| TN01 | Quét mã QR | Mở quét nhanh, camera hoặc ảnh thư viện, nhận diện VietQR tĩnh và động, tra tên người nhận, QR không hỗ trợ | [PRD](docs/TN01-quet-ma-qr/PRD.md) | [SRS](docs/TN01-quet-ma-qr/SRS.md) |
| TN02 | Chọn số tiền | Nhập số tiền (QR tĩnh) hoặc số tiền khoá (QR động), bằng chữ, gợi ý nhanh, nội dung, tài khoản nguồn, số dư, hạn mức | [PRD](docs/TN02-chon-so-tien/PRD.md) | [SRS](docs/TN02-chon-so-tien/SRS.md) |
| TN03 | Xác thực & xác nhận chuyển khoản | Xem lại, xác nhận, cảnh báo lừa đảo và chuyển trùng, PIN / sinh trắc / khuôn mặt, gửi lệnh | [PRD](docs/TN03-xac-thuc-xac-nhan/PRD.md) | [SRS](docs/TN03-xac-thuc-xac-nhan/SRS.md) |
| TN04 | Hoàn thành chuyển khoản | Đang xử lý, thành công, thất bại, chưa xác định, biên lai, chia sẻ, lưu người nhận, chi tiết giao dịch | [PRD](docs/TN04-hoan-thanh-chuyen-khoan/PRD.md) | [SRS](docs/TN04-hoan-thanh-chuyen-khoan/SRS.md) |

Thứ tự làm: TN01 → TN02 → TN03 → TN04, đúng thứ tự các bước của luồng.

## Luồng chuyển khoản QR

```mermaid
flowchart LR
  A["Trang chủ"] --> B["Quét mã QR, TN01"]
  B --> C["Tra tên người nhận"]
  C --> D["Chọn số tiền, TN02"]
  D --> E["Xem lại, Xác nhận, TN03"]
  E --> F["Xác thực PIN, sinh trắc, khuôn mặt, TN03"]
  F --> G["Hoàn thành, TN04"]
  G -->|Quét tiếp| B
```

## Vòng đời giao dịch (dùng chung cả đề)

```mermaid
stateDiagram-v2
  [*] --> NHAP : Quét QR, tra tên xong, TN01
  NHAP --> NHAP : Chọn số tiền, TN02
  NHAP --> DA_BO : Thoát, dừng ở cảnh báo, TN02, TN03
  NHAP --> BI_TU_CHOI : Rủi ro bị chặn, TN03
  NHAP --> CHO_XAC_THUC : Bấm Xác nhận, TN03
  CHO_XAC_THUC --> DA_BO : Hết 3 phút
  CHO_XAC_THUC --> BI_TU_CHOI : Sai xác thực quá số lần
  CHO_XAC_THUC --> DANG_XU_LY : Xác thực đúng, gửi lệnh, TN03
  DANG_XU_LY --> THANH_CONG : Ngân hàng nhận ghi có, TN04
  DANG_XU_LY --> THAT_BAI : Bị từ chối, tiền chưa trừ hoặc đã hoàn, TN04
  DANG_XU_LY --> CHUA_XAC_DINH : Quá 60 giây, TN04
  CHUA_XAC_DINH --> THANH_CONG : Đối soát tự động
  CHUA_XAC_DINH --> THAT_BAI : Đối soát, hoàn tiền
  THANH_CONG --> [*]
  THAT_BAI --> [*]
  DA_BO --> [*]
  BI_TU_CHOI --> [*]
```

## Bản đồ màn hình

```mermaid
flowchart LR
  S01["SCR-01 Trang chủ"] --> S02["SCR-02 Quét QR"]
  S02 --> S03["SCR-03 Chọn số tiền"]
  S03 --> S04["SCR-04 Xác nhận giao dịch"]
  S04 -->|Sửa| S03
  S04 --> S05["SCR-05 Xác thực"]
  S05 --> S06["SCR-06 Kết quả & biên lai"]
  S06 -->|Quét tiếp| S02
  S06 --> S07["SCR-07 Chi tiết giao dịch"]
  S07 -->|Chuyển lại| S03
```

## Bối cảnh chung

Chuyển khoản QR đã thay tiền mặt ở chợ, quán ăn, cửa hàng nhỏ. Người trả tiền cần **nhanh** vì có người chờ phía sau, nhưng cũng phải **chắc chắn**: QR có thể bị dán đè, số tiền có thể gõ nhầm, và kẻ gian có thể gọi điện dẫn dắt nạn nhân chuyển tiền. Bài toán thiết kế chính là cân bằng tốc độ và an toàn ở mỗi bước, đồng thời tuân thủ quy định xác thực của Ngân hàng Nhà nước.

| **Nhóm người dùng** | **Nhu cầu chính** |
| --- | --- |
| Người trả tiền ở cửa hàng | Quét, chọn tiền, xác thực, xong trong vài giây; kết quả to rõ |
| Người chia tiền với bạn bè | Quét ảnh QR từ tin nhắn, chia sẻ biên lai |
| Người mua online | QR có sẵn số tiền, nội dung |
| Người lớn tuổi, ít rành công nghệ | Đọc rõ người nhận và số tiền, được cảnh báo lừa đảo |

Ngoài phạm vi cả đề: mở tài khoản, đăng nhập và cài Smart OTP, đăng ký sinh trắc khuôn mặt (chỉ dẫn link), phía người nhận (tạo QR, thông báo tiền về), lịch sử giao dịch đầy đủ, tra soát, QR thanh toán qua cổng merchant, QR rút tiền ATM, QR quốc tế, chuyển khoản bằng số tài khoản (xem Đề 7).

## Quy ước dùng chung

- Tiền tệ VND, không có số lẻ; định dạng `250.000 ₫`; số tiền luôn kèm cách đọc bằng chữ ở màn nhập và xác nhận.
- Mã QR theo chuẩn VietQR (dựa trên chuẩn EMVCo); người nhận là tài khoản hoặc thẻ của ngân hàng thành viên Napas.
- Ngưỡng xác thực, hạn mức, phí là **cấu hình**; giá trị trong tài liệu là giá trị hiện hành theo Quyết định 2345/QĐ-NHNN.
- Số tổng đài hiển thị dạng `{số}`.
- Mã BR, MSG, UC đánh số liên tục trong cả đề: TN01 dùng BR-01–10, MSG-01–10, UC-01–02; TN02 dùng BR-11–20, MSG-11–18, UC-03; TN03 dùng BR-21–30, MSG-19–28, UC-04–05; TN04 dùng BR-31–38, MSG-29–35, UC-06–07.

## Danh sách màn hình

| **Mã** | **Màn hình** | **Bước** | **Mô tả chi tiết ở** |
| --- | --- | --- | --- |
| SCR-01 | Trang chủ (lối tắt Quét QR) | Quét mã QR | SRS TN01 |
| SCR-02 | Quét QR | Quét mã QR | SRS TN01 |
| SCR-03 | Chọn số tiền | Chọn số tiền | SRS TN02 |
| SCR-04 | Xác nhận giao dịch (gồm màn cảnh báo) | Xác thực & xác nhận | SRS TN03 |
| SCR-05 | Xác thực giao dịch | Xác thực & xác nhận | SRS TN03 |
| SCR-06 | Kết quả & biên lai | Hoàn thành | SRS TN04 |
| SCR-07 | Chi tiết giao dịch | Hoàn thành | SRS TN04 |

## Cấu trúc repo

```text
de04-chuyen-khoan-qr/
├── README.md
└── docs/
    ├── TN01-quet-ma-qr/                PRD.md · SRS.md
    ├── TN02-chon-so-tien/              PRD.md · SRS.md
    ├── TN03-xac-thuc-xac-nhan/         PRD.md · SRS.md
    └── TN04-hoan-thanh-chuyen-khoan/   PRD.md · SRS.md
```

## Cách dùng bộ tài liệu

1. Đọc README để nắm luồng 4 bước và vòng đời giao dịch.
2. Với mỗi bước: đọc PRD, rồi SRS. Thiết kế đủ trạng thái lỗi của từng bước (QR không hỗ trợ, thiếu số dư, cảnh báo, sai PIN, chưa xác định).
3. Ghép các màn thành một prototype liền mạch từ **SCR-01** tới **SCR-06**.
4. Thiết kế màn hình theo đúng mã SCR, mã MSG trong SRS để dễ đối chiếu khi review.
5. Sơ đồ viết bằng Mermaid, xem trực tiếp trên GitHub hoặc trình xem Markdown có hỗ trợ Mermaid.
