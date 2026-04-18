# SmartFoodStreet – Sơ đồ hệ thống (Mermaid)

> **Mở trong**: [Mermaid Live Editor](https://mermaid.live) để xem trực quan.

---

## 1. Use Case Diagram

```mermaid
flowchart TB
    Visitor(["👤 Visitor\n(Khách du lịch)"])
    Vendor(["👨‍💼 Vendor\n(Chủ quán)"])
    Admin(["🔐 Admin\n(Quản trị)"])
    System(["⚙️ System\n(Background Services)"])

    subgraph SFS ["🏪 SMARTFOODSTREET SYSTEM"]
        UC1["UC1: Truy cập khu phố ẩm thực\n(Public View)"]
        UC2["UC2: Kích hoạt Audio\ndựa trên vị trí GPS"]
        UC3["UC3: Xem chi tiết quán & menu"]
        UC4["UC4: Theo dõi & tìm quán từ GPS"]
        UC5["UC5: Quản lý tài khoản\n(Đăng ký / Đăng nhập / Cập nhật)"]
        UC6["UC6: Quản lý menu & hình ảnh\n(Tạo / Sửa / Xóa, Upload ảnh)"]
        UC7["UC7: Xem phân tích & thống kê\n(Analytics Dashboard)"]
        UC8["UC8: Quản lý hệ thống\n(Users, Permissions, Food Streets)"]
        UC9["UC9: Sinh âm thanh đa ngôn ngữ\n(Auto Translate + TTS)"]
    end

    Visitor --> UC1
    Visitor --> UC2
    Visitor --> UC3
    Visitor --> UC4

    Vendor --> UC5
    Vendor --> UC6
    Vendor --> UC7

    Admin --> UC8
    Admin --> UC5

    System --> UC9
    UC6 -.->|triggers| UC9
    UC2 -.->|uses| UC9
```

---

## 2. Activity Diagram – Kích hoạt Audio theo GPS (UC2)

```mermaid
flowchart TD
    Start(["▶ Visitor mở app"])
    A["App yêu cầu vị trí GPS"]
    B{GPS bật?}
    C["Hiển thị 'Vui lòng bật GPS'"]
    D["Tạo VisitSession\n(status = ACTIVE)"]
    E["App gửi GPS location\nmỗi 5–10 giây\nPOST /gps/check"]
    F["Server tính khoảng cách\nđến các stall lân cận"]
    G{Stall trong\ngeofence/radius?}
    H{Cooldown\n(120s) đã hết?}
    I["Bỏ qua – chờ đến lần\ngửi GPS tiếp theo"]
    J["Lấy audio URL\ntheo ngôn ngữ ưu tiên"]
    K{Audio đã\nCOMPLETED?}
    L["Trả về audio URL\n(preferred language)"]
    M["Trả về fallback audio\n(Vietnamese/English)"]
    N["App nhận response\n– tải hoặc cache audio"]
    O["Phát âm thanh\ngiới thiệu quán"]
    P["Log VisitEvent:\nAUDIO_START"]
    Q["Âm thanh kết thúc"]
    R["Log VisitEvent:\nAUDIO_COMPLETE"]
    S["Cập nhật cooldown\n(+120 giây)"]
    T(["⏹ Tiếp tục tracking GPS"])

    Start --> A --> B
    B -->|Không| C --> Start
    B -->|Có| D --> E --> F --> G
    G -->|Không| I --> E
    G -->|Có| H
    H -->|Chưa hết| I
    H -->|Đã hết| J --> K
    K -->|Có| L --> N
    K -->|Đang xử lý| M --> N
    N --> O --> P --> Q --> R --> S --> T --> E
```

---

## 3. Activity Diagram – Vendor tạo Stall & Audio đa ngôn ngữ (UC6)

```mermaid
flowchart TD
    Start(["▶ Vendor đăng nhập"])
    A["Mở form Tạo quán mới"]
    B["Nhập thông tin:\nTên, Thể loại, Tọa độ, Ảnh"]
    C["POST /stalls"]
    D["Validate dữ liệu\n(tọa độ, streetId...)"]
    E{Hợp lệ?}
    F["Báo lỗi về form"]
    G["Tạo Stall record\n(isActive = true)"]
    H["Tự tạo TriggerConfig mặc định\n(GEOFENCE, radius=30m, cooldown=120s)"]
    I["Trả về stallId → Vendor nhận\n'Tạo quán thành công'"]
    J["Vendor chọn 'Thêm bản dịch'"]
    K["Chọn ngôn ngữ & nhập TTS Script\nPOST /stall-translations"]
    L["Tạo StallTranslation\n(status = PENDING)"]
    M["Trả về 200 OK ngay lập tức\n(không block vendor)"]
    N["ASYNC: Phát hiện ngôn ngữ nguồn"]
    O{Cần dịch?}
    P["Gọi Google NMT API\n(Dịch văn bản)"]
    Q["Gọi Google TTS API\n(Tổng hợp giọng nói)"]
    R["Upload MP3 lên Cloudinary CDN"]
    S["UPDATE StallTranslation:\naudioUrl, audioHash, status=COMPLETED"]
    T["(Tùy chọn) Gửi webhook/email\n'Audio đã sẵn sàng!'"]
    End(["⏹ Hoàn tất"])

    Start --> A --> B --> C --> D --> E
    E -->|Không| F --> B
    E -->|Có| G --> H --> I --> J --> K --> L --> M
    M --> N --> O
    O -->|Có| P --> Q
    O -->|Không| Q
    Q --> R --> S --> T --> End
```

---

## 4. Activity Diagram – Vendor Đăng nhập & Xem Analytics (UC5 + UC7)

```mermaid
flowchart TD
    Start(["▶ Vendor mở app"])
    A["Nhập username & password"]
    B["POST /auth/login"]
    C["Query DB: tìm account"]
    D{Tài khoản\ntồn tại?}
    E["Trả về 401 Unauthorized"]
    F["BCrypt verify password"]
    G{Mật khẩu\nđúng?}
    H["Trả về 401 Unauthorized"]
    I["Tạo JWT token\n(userId, roles, permissions)"]
    J["Trả về JWT → App lưu vào\nsecure storage"]
    K["Vendor mở tab Analytics"]
    L["GET /visit-events?stallId=X&days=30\nAuthorization: Bearer token"]
    M["Validate JWT\n(@PreAuthorize)"]
    N{Token\nhợp lệ?}
    O["Trả về 403 Forbidden"]
    P["Query VisitEvent:\nthống kê 30 ngày"]
    Q["Tổng hợp số liệu:\nVisitors, Audio Plays,\nPeak Hours, Language Distribution"]
    R["Hiển thị Dashboard\ncho Vendor"]
    S{Vendor muốn\nexport?}
    T["Xuất báo cáo CSV/PDF"]
    End(["⏹ Kết thúc"])

    Start --> A --> B --> C --> D
    D -->|Không| E --> Start
    D -->|Có| F --> G
    G -->|Không| H --> Start
    G -->|Có| I --> J --> K --> L --> M --> N
    N -->|Không| O --> K
    N -->|Có| P --> Q --> R --> S
    S -->|Có| T --> End
    S -->|Không| End
```

---

## 5. Sequence Diagram – Visitor Nhận Audio Geofence (UC2)

```mermaid
sequenceDiagram
    actor V as Visitor
    participant App as Mobile App
    participant BE as Backend Server
    participant DB as MySQL Database
    participant CDN as Cloudinary CDN

    loop Mỗi 5–10 giây
        V->>App: Cung cấp tọa độ GPS
        App->>BE: POST /gps/check {sessionId, lat, lng, lang}
        BE->>DB: Spatial query: stalls trong radius
        DB-->>BE: Danh sách stalls lân cận
        BE->>DB: Kiểm tra cooldown & trigger config
        DB-->>BE: TriggerConfig + trạng thái cooldown
        alt Stall trong geofence & hết cooldown
            BE->>DB: Lấy audioUrl theo ngôn ngữ
            DB-->>BE: audioUrl (hoặc fallback)
            BE-->>App: 200 OK {triggeredStall, audioUrl, distance}
            App->>App: Kiểm tra audio cache (audioHash)
            alt Cache miss
                App->>CDN: GET audio file (MP3)
                CDN-->>App: ko_5.mp3 (245KB)
            end
            App-->>V: Phát âm thanh giới thiệu quán
            App->>BE: POST /visit-events {AUDIO_START, stallId}
            BE->>DB: INSERT VisitEvent (AUDIO_START)
            V->>App: Nghe xong
            App->>BE: POST /visit-events {AUDIO_COMPLETE}
            BE->>DB: INSERT VisitEvent (AUDIO_COMPLETE)
            BE->>DB: Cập nhật cooldown (+120s)
        else Không có geofence trigger
            BE-->>App: 200 OK {triggeredStall: null}
        end
    end
```

---

## 6. Sequence Diagram – Vendor Tạo Stall & Sinh Audio (UC6)

```mermaid
sequenceDiagram
    actor VN as Vendor
    participant App as Mobile App
    participant BE as Backend Server
    participant DB as MySQL Database
    participant GC as Google Cloud (NMT + TTS)
    participant CDN as Cloudinary CDN

    VN->>App: Tap "Tạo quán mới" & điền form
    App->>BE: POST /stalls {name, category, lat, lng, image}
    BE->>DB: INSERT Stall + auto-create TriggerConfig
    DB-->>BE: stallId = 5
    BE-->>App: 200 OK {stallId: 5}
    App-->>VN: "Tạo quán thành công!"

    VN->>App: Chọn ngôn ngữ Korean + nhập TTS script
    App->>BE: POST /stall-translations {stallId, lang:"ko", ttsScript}
    BE->>DB: INSERT StallTranslation (status=PENDING)
    BE-->>App: 200 OK {id: 999}
    App-->>VN: "Đang xử lý bản dịch..."

    Note over BE,CDN: ASYNC BACKGROUND TASKS

    BE->>GC: Translate Vietnamese → Korean (Google NMT)
    GC-->>BE: {translatedText: "안녕하세요..."}
    BE->>GC: Synthesize TTS audio (Google TTS)
    GC-->>BE: {audioData: binary MP3}
    BE->>CDN: Upload MP3 file
    CDN-->>BE: {audioUrl, fileSize, hash}
    BE->>DB: UPDATE StallTranslation {audioUrl, audioHash, status=COMPLETED}
    BE-->>App: Webhook "Korean audio ready!" (nếu bật)
    App-->>VN: Thông báo "Audio tiếng Hàn đã sẵn sàng!"
```

---

## 7. Sequence Diagram – Vendor Đăng nhập & Xem Analytics (UC5 + UC7)

```mermaid
sequenceDiagram
    actor VN as Vendor
    participant App as Mobile App
    participant BE as Backend Server
    participant DB as MySQL Database

    VN->>App: Nhập username & password
    App->>BE: POST /auth/login {username, password}
    BE->>DB: SELECT account WHERE username = ?
    DB-->>BE: Account record
    BE->>BE: BCrypt.verify(password, hash)
    alt Mật khẩu sai
        BE-->>App: 401 Unauthorized
        App-->>VN: "Tên đăng nhập hoặc mật khẩu không đúng"
    else Mật khẩu đúng
        BE->>BE: Generate JWT {userId, roles, permissions}
        BE-->>App: 200 OK {token: "eyJ..."}
        App->>App: Lưu token vào secure storage
        App-->>VN: Đăng nhập thành công

        VN->>App: Mở tab Analytics
        App->>BE: GET /visit-events?stallId=X&days=30\nAuthorization: Bearer token
        BE->>BE: Validate JWT + @PreAuthorize
        BE->>DB: SELECT analytics từ visit_events\n(30 ngày, grouped by type)
        DB-->>BE: Rows: total_visitors, audio_plays,\navg_time, peak_hours, lang_dist
        BE-->>App: 200 OK {analytics: {...}}
        App-->>VN: Dashboard: Visitors, Audio Plays,\nPeak Hours, Language Distribution
    end
```

---

## 8. Sequence Diagram – Zero-Latency Audio Fallback (UC6 Special)

```mermaid
sequenceDiagram
    actor V as Visitor (Korean)
    participant App as Mobile App
    participant BE as Backend Server
    participant DB as MySQL Database

    Note over V,DB: Scenario: Korean audio đang được generate (PENDING)

    V->>App: GPS vào geofence quán "Phở Huế 123"
    App->>BE: POST /gps/check {lang: "ko", sessionId, lat, lng}
    BE->>DB: Query StallTranslation WHERE stallId=5 AND lang="ko"
    DB-->>BE: {status: "PENDING", audioUrl: null}
    BE->>DB: Query fallback language (Vietnamese)
    DB-->>BE: {status: "COMPLETED", audioUrl: "vi_5.mp3"}
    BE-->>App: 200 OK {audioUrl: "vi_5.mp3", lang: "vi",\nmessage: "Korean đang xử lý, dùng tiếng Việt thay thế"}
    App-->>V: Phát audio tiếng Việt (fallback)

    Note over BE,DB: ≈ 15–30 giây sau — ASYNC hoàn tất

    BE->>DB: UPDATE StallTranslation ko: status=COMPLETED, audioUrl="ko_5.mp3"

    Note over V,DB: Lần sau Korean Visitor vào geofence

    V->>App: GPS vào geofence
    App->>BE: POST /gps/check {lang: "ko"}
    BE->>DB: Query StallTranslation WHERE lang="ko"
    DB-->>BE: {status: "COMPLETED", audioUrl: "ko_5.mp3"}
    BE-->>App: 200 OK {audioUrl: "ko_5.mp3", lang: "ko"}
    App-->>V: Phát audio tiếng Hàn ✅
```
