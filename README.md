# Chi nhánh In Tiến Thắng — bản triển khai thử nghiệm NAS

## Chức năng đã có
- Dashboard quản trị, phân hệ theo dõi đi làm, lọc phòng ban/tổ, mã công nhân và họ tên từ danh sách Excel đã cung cấp.
- API đọc chấm công, trạng thái theo ngày, nhập CSV, duyệt/hủy nghỉ phép qua API.
- PostgreSQL lưu bền vững, dữ liệu chấm công nhập lại không nhân đôi, cập nhật trang mỗi 30 giây.

## Cài trên Synology DS725+
1. Cài **Container Manager** và tạo thư mục riêng, ví dụ `/volume1/docker/tienthang`.
2. Chép toàn bộ thư mục dự án vào đó. Sao chép `.env.example` thành `.env`; thay hai giá trị bằng mật khẩu/khóa mạnh.
3. Trong Container Manager > Project, tạo project từ `docker-compose.yml` (build từ thư mục dự án). Nếu giao diện NAS không build được, dùng SSH chạy `docker compose up -d --build` trong thư mục.
4. Cấu hình Reverse Proxy DSM: tên miền HTTPS riêng → `http://127.0.0.1:8088` (nếu reverse proxy cùng NAS); cấp chứng chỉ TLS và chỉ cho truy cập qua HTTPS.
5. Kiểm tra website; trước khi mở cho người dùng thật phải bổ sung đăng nhập và phân quyền (bản hiện tại chưa có đăng nhập).

## Đồng bộ dữ liệu đã có trên NAS
Chưa xác định được định dạng/tên thư mục chấm công trên NAS nên **chưa có bộ đọc tự động trực tiếp**. Bản này nhận CSV UTF-8 với 2 cột `code,timestamp`; timestamp theo giờ địa phương `YYYY-MM-DD HH:MM:SS`, mã phải giữ số 0 đầu. Ví dụ:

```csv
code,timestamp
00009,2026-11-13 07:23:00
00011,2026-11-13 07:42:00
```

Gửi bằng `curl` từ máy có quyền truy cập (thay khóa):

```bash
curl -X POST 'https://TEN-MIEN-CUA-BAN/api/import-punches' -H 'X-Admin-Key: KHOA_BI_MAT' -F 'file=@chamcong.csv'
```

API `/api/overview?day=2026-11-13` trả về trạng thái và danh sách. Để tự động cập nhật từ NAS, cần biết thư mục/tên file, định dạng, cấu trúc cột và tần suất file được đồng bộ; sau đó thêm job đọc file và mapping cột. Không sửa file gốc trên NAS.

## Quy tắc
- Đi làm: có bản ghi đầu tiên không sau giờ bắt đầu ca.
- Vào muộn: bản ghi đầu tiên sau giờ bắt đầu ca.
- Nghỉ phép: chưa có bản ghi và được duyệt nghỉ qua API.
- Chưa chấm công: chưa có bản ghi, chưa duyệt nghỉ.
- Giờ chuẩn lấy từ cột lịch trong Excel; thiếu lịch dùng mặc định 07:30. **Đây là quy tắc tạm thời**: cần xác nhận ca đêm, nghỉ, ngày lễ, ngưỡng đi muộn và xử lý nhiều lượt quẹt trước khi dùng để đánh giá công nhân.
- 207 hồ sơ nhập từ bản gộp trước; 47 hồ sơ chưa có mã. Chưa xác minh trùng tên hoặc mã thiếu. Cần đối soát trước khi dùng thật.

## An toàn trước vận hành thật
- Hiện API xem danh sách chưa yêu cầu đăng nhập; **không đưa website ra Internet công khai** trước khi có xác thực, phân quyền và nhật ký truy cập.
- Các API ghi dữ liệu yêu cầu `X-Admin-Key`, chỉ dùng qua HTTPS.
- Không public port PostgreSQL; sao lưu volume `postgres_data` sang nơi độc lập với NAS.
- Mật khẩu trong `.env` phải giữ bí mật. Không đặt khóa trong frontend.
