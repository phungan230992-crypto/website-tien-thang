# CHI NHÁNH IN TIẾN THẮNG — Đồng bộ NAS V2

## Cài trên Synology DS725+
1. Cài **Container Manager** trên DSM; giải nén bộ cài vào thư mục dự án, ví dụ `/volume1/docker/tienthang`.
2. Xác định chính xác thư mục NAS đang chứa file xuất chấm công `.xlsx`. Mở `.env.example`, lưu thành `.env`, sửa `ATTENDANCE_HOST_PATH` thành đường dẫn thật; tạo mật khẩu mạnh riêng cho `DB_PASSWORD` và `ADMIN_API_KEY` (không dùng giá trị mẫu).
3. Trong Container Manager > Project, chọn thư mục dự án, dùng `docker-compose.yml`, Build/Start. Hoặc qua SSH: `docker compose up -d --build`.
4. Web chỉ lắng nghe `127.0.0.1:8088` trên NAS, chưa thể truy cập từ máy khác. Cấu hình DSM Reverse Proxy HTTPS có xác thực trước khi công khai; tuyệt đối không mở thẳng port PostgreSQL.
5. Đặt file xuất `D20261113085353.xlsx` vào thư mục đã gắn để thử; xem log container `sync`, sau đó kiểm tra `/api/schedule?day=2026-11-13` và `/api/overview?day=2026-11-13`.

## Ca làm
- Tuần gốc bắt đầu **thứ Hai 09/11/2026**: Thành 07:00–19:00, Huy 19:00–07:00 hôm sau.
- Đảo ca vào thứ Hai mỗi tuần. Ca đêm tính theo ngày bắt đầu ca.
- Văn phòng bắt đầu 07:30, giờ kết thúc chưa xác nhận.
- Vào muộn được tính ngay sau giờ bắt đầu, **chưa áp dụng thời gian ân hạn**.

## Dữ liệu
- Nhận file Excel mẫu cột `Mã N.Viên`, `Ngày`, `Vào1/Ra1`, `Vào2/Ra2`, `Vào3/Ra3`.
- Quét thư mục mỗi 30 giây; đọc lại khi file thay đổi; chống trùng bằng `(mã, thời điểm)`.
- Không suy đoán nghỉ phép từ ô `Công+` hoặc `K.hiệu`; nghỉ phép cần được nhân sự duyệt.
- Mã nhân viên phải duy trì dạng chuỗi có số 0 đầu.
- 7 phòng ban cố định. Danh sách 207 hồ sơ từ bản gộp Excel cũ **chưa xác minh đủ mã và phòng ban**; cần rà soát trước khi vận hành.

## Giới hạn cần xử lý trước khi đưa vào sử dụng thật
- **Chưa có xác thực người dùng và phân quyền cho API đọc**, không được đưa lên Internet cho tới khi bổ sung đăng nhập và phân quyền.
- Báo cáo xuất từ máy chấm công có thể là ảnh chụp tại thời điểm xuất; nếu không xuất lại, web không có bản ghi mới. Chỉ hiển thị thời điểm đồng bộ file, không khẳng định kết nối trực tiếp máy.
- Mẫu file có thể chứa mã nghỉ, công cộng thêm, ca đổi đột xuất. Chưa tự suy diễn trạng thái từ các cột này.
- Cần xử lý trường hợp ra/vào không đúng cột, ca đêm thiếu bản ghi, bản ghi qua nửa đêm, nghỉ lễ, nghỉ bù, và đổi ca ngoại lệ.
- Chưa có cơ chế đối soát xóa/sửa bản ghi nếu file nguồn bị chỉnh lại; bộ đồng bộ hiện thêm bản ghi mới, không xóa bản ghi cũ.
- Cần sao lưu DB, giới hạn quyền NAS, quản lý mật khẩu và audit log trước khi chạy sản xuất.
