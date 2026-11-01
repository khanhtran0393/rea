# Bản ghi Mạng Web Lịch sử (Historical Web Network Captures)

REA cung cấp khả năng nạp, phân tích và trích xuất bằng chứng sâu từ các tệp ghi lưu lượng mạng lịch sử mà không cần kết nối mạng trực tiếp hoặc chạy lại ứng dụng.

## Định dạng và Hồ sơ Thượng nguồn (Formats and Upstream Profiles)

REA hỗ trợ nhiều định dạng lưu trữ lưu lượng mạng phổ biến:
- **HAR (HTTP Archive)**: Tiêu chuẩn mở được xuất từ Chrome, Firefox, Edge, Safari hoặc Charles Proxy.
- **SAZ (Session Archive Zip)**: Định dạng lưu trữ phiên của Fiddler Classic và Fiddler Everywhere.
- **PCAP / PCAPNG**: Định dạng bắt gói tin mạng thô của Wireshark và tcpdump.
- **Qlog**: Định dạng bản ghi nhật ký có cấu trúc cho giao thức HTTP/3 và QUIC.

## Đọc Bản ghi Fiddler (`.saz`)

REA tự động giải nén cấu trúc tệp ZIP của `.saz`:
- Đọc các tệp `_c.txt` (yêu cầu máy khách) và `_s.txt` (phản hồi máy chủ).
- Trích xuất siêu dữ liệu thời gian, độ trễ và cờ trạng thái từ tệp XML `_m.xml`.

```sh
rea inspect-capture ./session.saz --json
```

## Thân thông điệp Protobuf và gRPC (Protobuf and gRPC bodies)

Khi phát hiện các yêu cầu hoặc phản hồi nhị phân thuộc định dạng Protocol Buffers hoặc gRPC:
- REA tự động phân tích cú pháp các trường số học, chuỗi và cấu trúc lồng nhau (schemaless protobuf decoding).
- Nếu bạn có tệp định nghĩa `.proto` gốc, bạn có thể truyền vào để giải mã đầy đủ tên trường:

```sh
# Biên dịch lược đồ của bạn bằng protoc với các import cục bộ:
protoc --descriptor_set_out=./schema.desc ./service.proto
rea inspect-capture ./grpc.har --proto-descriptors ./schema.desc --json
```

## Fiddler Everywhere MCP và Kỹ năng Công khai

REA có thể hoạt động độc lập hoặc bổ trợ cho các kỹ năng Fiddler Everywhere, cung cấp khả năng tìm kiếm nâng cao theo biểu thức chính quy, trích xuất mã thông báo và phân tích tương quan phiên.

## Đọc Bằng chứng (Reading the Evidence)

Đầu ra của công cụ phân tích mạng bao gồm:
- **Danh mục Điểm cuối (Endpoints inventory)**: Danh sách đầy đủ các URL duy nhất, phân nhóm theo tên miền và phương thức HTTP.
- **Tham số và Tiêu đề**: Phân tích tần suất và phạm vi giá trị của các tham số truy vấn (`query params`) và tiêu đề HTTP.
- **Luồng dữ liệu nhạy cảm**: Tự động phát hiện các chuỗi có định dạng khóa bí mật, JWT token, hoặc thông tin cá nhân (PII).

## Bề mặt API Dẫn xuất (Derived API Surface)

Từ tập hợp các yêu cầu mạng được ghi lại, REA có thể tự động suy luận và sinh ra đặc tả OpenAPI (Swagger) sơ bộ:
- Cấu trúc đường dẫn URL và các tham số biến động (`/users/{id}`).
- Lược đồ JSON của thân yêu cầu và phản hồi.
- Các mã trạng thái HTTP quan sát được (200, 201, 400, 401, 404, 500).

```sh
rea derive-openapi ./capture.har --output ./openapi.yaml
```

## Manifest Môi trường và Trạng thái Môi trường

Bản ghi mạng là nguồn nguyên liệu chính để xây dựng các gói môi trường tác tử (Agent Environments):
- Trích xuất dữ liệu phản hồi để nạp vào `state.json`.
- Thiết lập các tuyến đường phát lại tự động trong `manifest.json`.

## Độ bao phủ của Môi trường (Environment Coverage)

Đánh giá tỷ lệ các tuyến API trong tài liệu đã được ghi nhận trong bản lưu lượng mạng, giúp xác định các kịch bản kiểm thử chưa được bao phủ.

## Ranh giới Tài nguyên và Vòng đời

- Hỗ trợ xử lý tệp HAR/PCAP dung lượng lớn (lên tới vài GB) thông qua cơ chế đọc luồng (streaming parser), tránh nạp toàn bộ tệp vào bộ nhớ RAM cùng lúc.
- Giới hạn bộ đệm giải nén tệp nén gzip/brotli để phòng chống tấn công nén (zip bomb).

## Tọa độ Bị loại trừ và Lỗi Đọc (Excluded coordinates and read failures)

Các gói tin bị phân mảnh không hoàn chỉnh, phiên TLS bị mã hóa không có khóa giải mã, hoặc luồng dữ liệu bị cắt ngắn sẽ được ghi nhận rõ ràng vào danh sách lỗi đọc trong Bằng chứng thay vì làm dừng đột ngột quá trình phân tích.

## Lưu giữ Qlog của aioquic (Retained aioquic qlog)

Hỗ trợ phân tích chuyên sâu cho các phiên giao tiếp HTTP/3 qua QUIC:
- Theo dõi các sự kiện bắt tay mã hóa TLS 1.3, mất gói tin và kiểm soát tắc nghẽn.
- Phân tích các khung dữ liệu HTTP/3 (HEADERS, DATA, SETTINGS).

### Chế độ xem Qlog Tập trung (Focused qlog views)

Cho phép lọc và trích xuất chỉ các luồng dữ liệu cụ thể (stream-specific events) để điều tra sự cố hiệu năng hoặc lỗi kết nối.
