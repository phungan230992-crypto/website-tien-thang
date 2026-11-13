# Chi nhánh In Tiến Thắng — mã nguồn V2

Bản mã nguồn để lưu trong **GitHub Private Repository**. Chưa triển khai production; chưa đủ xác thực cho website công khai.

## Cấu hình
1. Sao chép `.env.example` thành `.env` trên NAS (không commit `.env`).
2. Thay DB_PASSWORD và ADMIN_API_KEY bằng giá trị bí mật mạnh.
3. Cấu hình ATTENDANCE_HOST_PATH tới thư mục NAS chứa file Excel.
4. Kiểm tra quyền truy cập thư mục, quy tắc ca, dữ liệu và bảo mật trước khi triển khai.

**Dữ liệu công nhân thật đã được loại khỏi gói GitHub**: `app/workers.json` là `[]`. Không upload file chấm công, dữ liệu nhân sự, mật khẩu, backup hoặc database lên GitHub.

## Lịch ca
Tuần 09/11/2026: Ca Thành ngày 07:00–19:00; Ca Huy đêm 19:00–07:00 hôm sau. Đổi thứ Hai hằng tuần. Văn phòng bắt đầu 07:30.
