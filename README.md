## Hệ thống Thu học phí Online (ASP.NET Core 9 + EF Core + SQL Server)

Dự án web quản lý và thu học phí online cho sinh viên, gồm backend ASP.NET Core 9, EF Core (SQL Server), xác thực JWT và tích hợp đăng nhập Google/Facebook. Frontend tĩnh phục vụ từ `wwwroot` (Razor Pages + HTML/JS/CSS).

### Tính năng chính
- Quản lý khoa (`Department`), lớp (`Class`), sinh viên (`Student`), học phần (`Course`).
- Cấu trúc học phí (`FeeStructure`), thu học phí (`StudentFee`, `StudentFeeDetail`, `Payment`).
- Xuất hóa đơn (`Invoice`), thông báo (`Notification`).
- Xác thực/ủy quyền bằng JWT; hỗ trợ đăng nhập Google/Facebook.
- Gửi email (SMTP) cho xác nhận email, quên mật khẩu.
- Tài liệu API qua Swagger UI.

### Công nghệ
- .NET SDK 9, ASP.NET Core 9
- Entity Framework Core 9 (SQL Server)
- Swashbuckle (Swagger)
- BCrypt/Hash SHA-256 (mã hóa mật khẩu trong các phần khác nhau của hệ thống)

### Cấu trúc thư mục chính
- `WebProject/`
  - `Program.cs`: cấu hình dịch vụ, JWT, CORS, Swagger, seeding dev, routing, static files.
  - `Models/Entities/`: các entity như `User`, `Student`, `Department`, `Class`, `FeeStructure`, `Payment`, `Invoice`...
  - `Models/ApplicationDbContext.cs`: DbContext, cấu hình quan hệ, khóa duy nhất, hạn chế xóa cascade.
  - `Controllers/`: API cho phòng ban, lớp, học phí, thanh toán, hóa đơn, thông báo, v.v.
  - `Authentication/Controllers` và `Authentication/Services`: Auth (đăng ký/đăng nhập, xác nhận email, quên mật khẩu, Google/Facebook).
  - `Migrations/`: các migration EF Core.
  - `wwwroot/`: giao diện tĩnh (HTML/JS/CSS) cho admin/sinh viên (dashboard, profile, tuition, payment...).
  - `DbInitializer.cs`: migrate DB và seed dữ liệu danh mục cơ bản (Departments, Classes).

### Yêu cầu hệ thống
- .NET SDK 9: cài từ `https://dotnet.microsoft.com/download/dotnet/9.0`.
- SQL Server (local hoặc cloud). Có thể dùng SQL Server Express + `TrustServerCertificate=True` khi phát triển.
- Công cụ EF Core CLI:
```bash
dotnet tool install --global dotnet-ef
```

### Cấu hình môi trường (khuyên dùng User Secrets trong dev)
Không commit bí mật vào repo. Thay vì sửa `appsettings.json`, hãy đặt secrets:
```bash
cd WebProject
dotnet user-secrets init

# Kết nối DB
dotnet user-secrets set "ConnectionStrings:DefaultConnection" "Server=.;Database=OnlineFeeDb;Trusted_Connection=True;TrustServerCertificate=True;MultipleActiveResultSets=True"

# JWT
dotnet user-secrets set "Jwt:Key" "your-256-bit-secret"
dotnet user-secrets set "Jwt:Issuer" "OnlineFeePaymentSystem"
dotnet user-secrets set "Jwt:Audience" "Students"

# SMTP (ví dụ Gmail)
dotnet user-secrets set "SmtpSettings:Host" "smtp.gmail.com"
dotnet user-secrets set "SmtpSettings:Port" "587"
dotnet user-secrets set "SmtpSettings:Username" "your@gmail.com"
dotnet user-secrets set "SmtpSettings:Password" "app-password"
dotnet user-secrets set "SmtpSettings:FromEmail" "your@gmail.com"
dotnet user-secrets set "SmtpSettings:FromName" "Online Fee Payment System"

# OAuth
dotnet user-secrets set "Authentication:Google:ClientId" "..."
dotnet user-secrets set "Authentication:Google:ClientSecret" "..."
dotnet user-secrets set "Authentication:Facebook:AppId" "..."
dotnet user-secrets set "Authentication:Facebook:AppSecret" "..."

# App URL (dùng để tạo link xác nhận/quên mật khẩu)
dotnet user-secrets set "AppUrl" "http://localhost:5083"
```

Lưu ý: file `WebProject/appsettings.json` hiện có ví dụ cấu hình thật (DB, SMTP). Khi triển khai thực tế, hãy thay thế bằng biến môi trường/secret riêng của bạn.

### Khởi chạy dự án (local)
```bash
cd WebProject
dotnet restore
dotnet ef database update   # tạo DB theo migrations
dotnet run                  # chạy ở http://localhost:5083 (https: 7274)
```

- Swagger UI: `http://localhost:5083/swagger`
- Trang chủ sẽ tự redirect tới `index.html` trong `wwwroot`.

### Seed dữ liệu
- Khi chạy ở môi trường Development, hệ thống sẽ:
  - Thực hiện `Database.Migrate()` tự động (trong `DbInitializer`).
  - Seed danh mục `Departments`, `Classes` mẫu.
- `Program.cs` có thêm logic seed dữ liệu demo (users/sinh viên) để thuận tiện thử nghiệm.

### Ghi chú bảo mật
- Không commit khóa JWT, mật khẩu SMTP, client secret OAuth.
- Dùng User Secrets (dev) hoặc biến môi trường/KeyVault (prod).
- Cấu hình CORS: hiện cho phép mọi origin trong dev (`AllowAll`). Hãy siết chặt trong production.

### Build & Deploy
- Thiết lập biến môi trường `ASPNETCORE_ENVIRONMENT=Production`.
- Cấu hình chuỗi kết nối, JWT, SMTP, OAuth bằng biến môi trường.
- Publish:
```bash
cd WebProject
dotnet publish -c Release -o out
```
Triển khai thư mục `out/` lên máy chủ Windows/IIS, Linux/Nginx, hoặc dịch vụ cloud.

### Liên hệ / Đóng góp
Mở issue hoặc PR để đề xuất tính năng/sửa lỗi. Hãy tuân theo chuẩn mã hóa, đặt biến rõ nghĩa và tránh thay đổi không liên quan.
