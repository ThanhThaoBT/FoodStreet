# ACTIVITY DIAGRAMS (Mermaid)

---

## 1. UC1 - Bắt đầu tour

```mermaid
flowchart TD
    A[Start] --> B[Mở ứng dụng]
    B --> C[Nhấn Bắt đầu tour]
    C --> D{Có quyền GPS?}
    D -- Không --> E[Yêu cầu cấp quyền]
    E --> D
    D -- Có --> F[Lấy vị trí hiện tại]
    F --> G[Tạo visit_session]
    G --> H[Bắt đầu GPS tracking]
    H --> I[Hiển thị bản đồ]
    I --> J[End]
```

---

## 2. UC2 - Nhận audio tự động (Geofencing)

```mermaid
flowchart TD
    A[Start] --> B[Nhận GPS định kỳ]
    B --> C[Lấy danh sách gian hàng]
    C --> D[Loop từng gian hàng]
    D --> E[Check distance]
    E --> F[Check radius]
    F --> G[Check cooldown]
    G --> H{Hợp lệ?}
    H -- Không --> D
    H -- Có --> I[Chọn theo priority]
    I --> J[Call Audio Service]
    J --> K{audioStatus}
    K -- PENDING/PROCESSING --> L[Trả PROCESSING]
    K -- COMPLETED --> M[Trả audioUrl]
    M --> N[Phát audio]
    N --> O[Log AUDIO_START]
    O --> P[Log AUDIO_COMPLETE]
    P --> Q[End]
```

---

## 3. UC3 - Quét QR

```mermaid
flowchart TD
    A[Start] --> B[Mở chức năng QR]
    B --> C[Quét QR]
    C --> D[Lấy stall_id]
    D --> E[Load thông tin gian hàng]
    E --> F[Hiển thị chi tiết]
    F --> G[Call Audio API]
    G --> H{audioStatus}
    H -- COMPLETED --> I[Phát audio]
    H -- PROCESSING --> J[Chờ / retry]
    I --> K[Log AUDIO_START]
    K --> L[End]
```

---

## 4. UC4 - Xem bản đồ

```mermaid
flowchart TD
    A[Start] --> B[Mở bản đồ]
    B --> C[Load danh sách gian hàng]
    C --> D[Hiển thị marker]
    D --> E[Click marker]
    E --> F[Hiển thị chi tiết]
    F --> G{User phát audio?}
    G -- Có --> H[Call Audio API]
    G -- Không --> I[End]
    H --> I
```

---

## 5. UC6 - Chuyển đổi ngôn ngữ

```mermaid
flowchart TD
    A[Start] --> B[Chọn ngôn ngữ]
    B --> C[Update UI]
    C --> D[Xác định stall hiện tại]
    D --> E[Call Audio API theo ngôn ngữ]
    E --> F{audioStatus}
    F -- COMPLETED --> G[Phát audio]
    F -- PROCESSING --> H[Chờ / retry]
    G --> I[End]
    H --> I
```

---

## 6. UC8 - Quản lý nội dung TTS (Vendor)

```mermaid
flowchart TD
    A[Start] --> B[Đăng nhập]
    B --> C[Chọn gian hàng]
    C --> D[Chọn ngôn ngữ]
    D --> E[Nhập nội dung]
    E --> F[Lưu]
    F --> G[Reset audioStatus = PENDING]
    G --> H[End]
```

---

## 7. UC9 - Quản lý gian hàng + Geofencing

```mermaid
flowchart TD
    A[Start] --> B[Đăng nhập Admin]
    B --> C[Chọn quản lý gian hàng]
    C --> D[Thêm / sửa / xoá]
    D --> E[Nhập thông tin gian hàng]
    E --> F[Nhập cấu hình geofencing]
    F --> G[Validate]
    G --> H[Lưu stalls]
    H --> I[Lưu trigger_config]
    I --> J[End]
```

---

## 8. UC11 - Quản lý QR

```mermaid
flowchart TD
    A[Start] --> B[Chọn quản lý QR]
    B --> C[Chọn gian hàng]
    C --> D[Tạo QR]
    D --> E[Encode stall_id / URL]
    E --> F[Lưu QR]
    F --> G[Tải xuống]
    G --> H[End]
```

---

## 9. UC13 - Quản lý tài khoản

```mermaid
flowchart TD
    A[Start] --> B[Admin đăng nhập]
    B --> C[Quản lý tài khoản]
    C --> D[Thêm / sửa / xoá]
    D --> E[Nhập thông tin + role]
    E --> F[Validate]
    F --> G[Lưu DB]
    G --> H[End]
```

---

## 10. UC14 - Xem thống kê hệ thống

```mermaid
flowchart TD
    A[Start] --> B[Truy cập dashboard]
    B --> C[Hiển thị số liệu tổng]
    C --> D[Chọn loại thống kê]
    D --> E[Hiển thị biểu đồ]
    E --> F{Lọc thời gian?}
    F -- Có --> G[Filter dữ liệu]
    F -- Không --> H[Hiển thị mặc định]
    G --> H
    H --> I[End]
```

---

## 11. Activity Diagram tổng (Flow chính hệ thống)

```mermaid
flowchart TD
    A[Start] --> B[Client gửi GPS]
    B --> C[Server nhận lat/lng]
    C --> D[Query gian hàng]
    D --> E[Loop từng stall]
    E --> F[Check radius]
    F --> G[Check cooldown]
    G --> H[Chọn theo priority]
    H --> I[Call Audio Service]
    I --> J{audioStatus}
    J -- PENDING --> K[Set PROCESSING]
    K --> L[Async TTS]
    L --> M[Upload Cloudinary]
    M --> N[Update DB COMPLETED]
    J -- COMPLETED --> O[Trả audioUrl]
    O --> P[Client phát audio]
    P --> Q[End]
```
