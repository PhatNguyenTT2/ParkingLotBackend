# Parking Lot Management System - Backend API

Hệ thống quản lý bãi đỗ xe với đầy đủ tính năng quản lý thẻ, đăng ký, định giá và điều hành ra vào.

## 📋 Tổng quan Models

### 🔵 Phase 1: Core Entities (Nền tảng)

#### Person
- **Mục đích**: Quản lý thông tin cá nhân cơ bản
- **ID**: `PER0001`
- **Chức năng**: Lưu trữ thông tin người (họ tên, SĐT, email, giới tính)
- **Quan hệ**: Là base cho Customer và Employee

#### VehicleType
- **Mục đích**: Phân loại loại phương tiện
- **ID**: `VTP0001`
- **Chức năng**: Định nghĩa các loại xe (xe máy, ô tô, xe tải, xe đạp)
- **Quan hệ**: Được tham chiếu bởi Vehicle và các quy tắc định giá

#### CardCategory
- **Mục đích**: Phân loại thẻ đỗ xe
- **ID**: `CCG0001`
- **Chức năng**: Định nghĩa loại thẻ (lượt, tháng, quý, năm)
- **Quan hệ**: Được tham chiếu bởi Card, CardPrice và các quy tắc định giá

---

### 👥 Phase 2: User Management (Quản lý người dùng)

#### Customer
- **Mục đích**: Quản lý thông tin khách hàng
- **ID**: `CUS0001` (kế thừa Person.ID)
- **Chức năng**: Lưu trữ ngày đăng ký, trạng thái khách hàng
- **Trạng thái**: ACTIVE, INACTIVE, SUSPENDED
- **Quan hệ**: Mở rộng từ Person, tham chiếu bởi CardPurchaseInvoice và Subscription

#### Employee
- **Mục đích**: Quản lý thông tin nhân viên
- **ID**: `EMP0001` (kế thừa Person.ID)
- **Chức năng**: Lưu loại nhân viên, ngày bắt đầu, mức lương, trạng thái
- **Loại**: STAFF, MANAGER, ADMIN
- **Trạng thái**: ACTIVE, INACTIVE, ON_LEAVE, TERMINATED
- **Quan hệ**: Mở rộng từ Person, tham chiếu bởi StaffAccount và AdminAccount

#### StaffAccount
- **Mục đích**: Tài khoản cho nhân viên điều hành (entry/exit)
- **ID**: `STA0001`
- **Chức năng**: Xác thực bằng mã PIN cho các thao tác ra vào
- **Trạng thái**: ACTIVE, LOCKED, EXPIRED
- **Quan hệ**: One-to-one với Employee

#### AdminAccount
- **Mục đích**: Tài khoản quản trị hệ thống
- **ID**: `ADM0001`
- **Chức năng**: Xác thực bằng username/password cho admin panel
- **Bảo mật**: Mã hóa password bằng bcrypt
- **Quan hệ**: One-to-one với Employee

---

### 🚗 Phase 3: Vehicle & Card Management

#### Vehicle
- **Mục đích**: Quản lý thông tin phương tiện
- **ID**: `VEH0001`
- **Chức năng**: Lưu biển số, loại xe, chủ sở hữu, trạng thái
- **Trạng thái**: ACTIVE, BLOCKED, LOST
- **Validation**: Định dạng biển số xe Việt Nam
- **Quan hệ**: Tham chiếu VehicleType, được tham chiếu bởi Card, Subscription, EntrySession

#### Card
- **Mục đích**: Quản lý thẻ đỗ xe RFID
- **ID**: `CRD0001`
- **Chức năng**: Lưu UID thẻ, liên kết với loại thẻ, người sở hữu, xe
- **Đặc biệt**: UID cho quét thẻ RFID
- **Quan hệ**: Tham chiếu CardCategory, Person, Vehicle; được dùng trong Invoice, Subscription, EntrySession, CardReturn

#### CardPrice
- **Mục đích**: Lịch sử giá thẻ
- **ID**: `CPR0001`
- **Chức năng**: Lưu giá theo thời gian, lý do thay đổi
- **Pattern**: Immutable - không cập nhật, chỉ thêm mới
- **Quan hệ**: Tự tham chiếu (chuỗi lịch sử giá)

---

### 💰 Phase 4: Pricing Rules (Quy tắc định giá)

#### SubscriptionType
- **Mục đích**: Định nghĩa loại đăng ký
- **ID**: `SUB0001`
- **Chức năng**: Xác định thời hạn (MONTHLY: 30 ngày, QUARTERLY: 90 ngày, YEARLY: 365 ngày)
- **Ví dụ**: Tháng, Quý, Năm

#### SinglePricingRule
- **Mục đích**: Quy tắc giá theo giờ/ngày
- **ID**: `SPR0001`
- **Chức năng**: Định giá cho thẻ lượt, lưu lịch sử thay đổi
- **Pattern**: Immutable pricing history
- **Quan hệ**: Tự tham chiếu (lịch sử giá)

#### SubscriptionPricingRule
- **Mục đích**: Container cho giá đăng ký
- **ID**: `SPS0001`
- **Chức năng**: Nhóm định giá theo loại thẻ + loại xe + loại đăng ký
- **Unique**: Composite key (CardCategoryID + VehicleTypeID + SubscriptionTypeID)
- **Quan hệ**: Được tham chiếu bởi SubscriptionPricingRuleDetail

#### SubscriptionPricingRuleDetail
- **Mục đích**: Lịch sử giá đăng ký chi tiết
- **ID**: `SPD0001`
- **Chức năng**: Lưu giá thực tế, ngày hiệu lực, lý do thay đổi
- **Pattern**: Immutable price history
- **Quan hệ**: Tham chiếu SubscriptionPricingRule, tự tham chiếu (lịch sử)

---

### 🧾 Phase 5: Sales & Invoicing (Bán hàng & Hóa đơn)

#### CardPurchaseInvoice
- **Mục đích**: Quản lý hóa đơn mua thẻ
- **ID**: `INV0001`
- **Chức năng**: Lưu thông tin mua thẻ, tổng tiền, nhân viên xử lý
- **Trạng thái**: PENDING, COMPLETED, CANCELLED, PARTIALLY_RETURNED, FULLY_RETURNED
- **Logic**: Tính tổng tiền từ details, cập nhật trạng thái theo returns
- **Quan hệ**: Tham chiếu Customer, được tham chiếu bởi CardPurchaseDetail, CardReturn, ReturnBatch

#### CardPurchaseDetail
- **Mục đích**: Chi tiết hóa đơn (quan hệ nhiều-nhiều)
- **Composite Key**: (InvoiceID, CardID)
- **Chức năng**: Liên kết Invoice với Card, lưu giá mua, ghi chú
- **Pattern**: Không có ID riêng, dùng composite key
- **Note**: Có thể embedded trong Invoice hoặc collection riêng

---

### 🔄 Phase 6: Returns System (Hệ thống trả thẻ)

#### CardReturn
- **Mục đích**: Quản lý việc trả thẻ từng cái
- **ID**: `CRT0001`
- **Chức năng**: Xử lý trả thẻ, tính hoàn tiền, lưu lý do
- **Lý do**: NOT_USED, DAMAGED, DEFECTIVE, CHANGED_MIND
- **Trạng thái**: PENDING, APPROVED, REJECTED, PROCESSED
- **Phương thức hoàn**: CASH, BANK_TRANSFER, EWALLET, CREDIT
- **Logic**: Tính hoàn tiền dựa trên sử dụng, cập nhật trạng thái invoice
- **Unique**: Một thẻ chỉ được trả một lần

#### ReturnBatch
- **Mục đích**: Tổng hợp returns theo hóa đơn
- **ID**: `RTB0001`
- **Chức năng**: Tập hợp tất cả returns của một invoice
- **Trạng thái hoàn**: PARTIAL, FULL, ZERO, FULL_PROCESSED, PARTIAL_PROCESSED
- **Pattern**: Tự động tính toán từ CardReturn records
- **Quan hệ**: One-to-one với Invoice

---

### 📅 Phase 7: Subscription Management (Quản lý đăng ký)

#### Subscription
- **Mục đích**: Quản lý đăng ký dài hạn
- **ID**: `SSN0001`
- **Chức năng**: Quản lý đăng ký xe theo tháng/quý/năm
- **Logic**: 
  - Tự động tính endDate từ startDate + duration
  - Hỗ trợ tạm ngưng/tiếp tục
  - Kiểm tra hợp lệ khi ra vào
- **Quan hệ**: Tham chiếu Customer, Card, Vehicle, SubscriptionPricingRuleDetail

---

### 🚪 Phase 8: Operations (Điều hành ra vào)

#### EntrySession
- **Mục đích**: Quản lý phiên ra vào bãi đỗ
- **ID**: `ENT0001`
- **Chức năng**: 
  - Entry: Tạo session, validate thẻ/đăng ký
  - Exit: Tính phí, xử lý thanh toán
  - Xử lý mất thẻ, giảm giá
- **Trạng thái**: IN_PARKING, EXITED, LOST_TICKET, CANCELLED
- **Lý do giảm**: STAFF_FREE, SUBSCRIPTION, PROMO, MANUAL_OVERRIDE
- **Unique**: (CardID, EntryTime)
- **Quan hệ**: Tham chiếu Vehicle, Card, Subscription, StaffAccount

---

## 🔑 Key Features

### ID Generation
- Tất cả entities có ID tự động sinh: `[PREFIX][4-DIGIT-SEQUENCE]`
- Ví dụ: PER0001, VEH0123, CRD0456

### Immutable Price History
- CardPrice, SinglePricingRule, SubscriptionPricingRuleDetail
- Không update giá cũ, chỉ thêm bản ghi mới
- Tham chiếu đến version trước đó

### Status Management
- Hầu hết entities có trường `status` hoặc `isActive`
- Hỗ trợ soft delete và workflow

### Timestamps
- Tất cả models có `createdAt` và `updatedAt` tự động

---

## 🛠 Technical Stack

- **Database**: MongoDB with Mongoose ODM
- **Authentication**: bcrypt for passwords, PIN codes for staff
- **Validation**: Mongoose validators + custom business rules
- **Patterns**: MVC, Repository Pattern
- **Security**: Role-based access control (RBAC)

---

## 📊 Total Models: 20

- **Phase 1**: 3 models (Foundation)
- **Phase 2**: 4 models (User Management)
- **Phase 3**: 3 models (Vehicle & Card)
- **Phase 4**: 4 models (Pricing Rules)
- **Phase 5**: 2 models (Sales & Invoicing)
- **Phase 6**: 2 models (Returns)
- **Phase 7**: 1 model (Subscription)
- **Phase 8**: 1 model (Operations)
