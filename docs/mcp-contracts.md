# Hợp đồng Thời gian chạy MCP (MCP Runtime Contracts)

Tài liệu này xác định các cam kết kiến trúc, lược đồ dữ liệu và hợp đồng giao tiếp giữa máy chủ Model Context Protocol (MCP) của REA và các ứng dụng khách (clients) hoặc tác tử AI (agents).

## Kết quả Công cụ (Tool Results)

Mọi công cụ trong máy chủ REA MCP đều trả về phản hồi tuân thủ cấu trúc nội dung MCP chuẩn (`CallToolResult`):
- `content`: Danh sách các khối văn bản (`text`), hình ảnh (`image`), hoặc tài nguyên đính kèm (`resource`).
- `isError`: Cờ boolean cho biết lệnh thất bại hay thành công.
- `evidence`: Khối siêu dữ liệu có cấu trúc chứa Bằng chứng được lưu giữ (Retained Evidence), ID phiên và các số liệu kỹ thuật.

### Hạn mức Tài nguyên Xác thực Đầu vào

REA áp dụng quy trình xác thực nghiêm ngặt cho mọi tham số đầu vào bằng JSON Schema / Zod:
- Phát hiện lỗi cú pháp và thiếu trường bắt buộc ngay lập tức trước khi phân bổ tài nguyên tính toán.
- Giới hạn kích thước tải dữ liệu yêu cầu để tránh tấn công từ chối dịch vụ bộ nhớ.

### Khắc phục Lỗi và Quyền sở hữu Thời hạn (Deadline ownership)

- Mỗi yêu cầu MCP được gán một thời hạn tối đa (deadline/timeout).
- Nếu một tác vụ phân tích tốn kém (ví dụ: Ghidra decompile) sắp chạm ngưỡng thời gian chờ, REA chủ động hủy bỏ tiến trình con một cách an toàn và trả về kết quả phân tích một phần kèm theo thông báo trạng thái rõ ràng, thay vì để kết nối bị ngắt đột ngột.

### Nội dung Mục tiêu Không đáng tin cậy

Mọi chuỗi ký tự, mã máy hoặc dữ liệu trích xuất từ tệp nhị phân mục tiêu đều được xử lý như dữ liệu không đáng tin cậy (untrusted):
- Tự động làm sạch các ký tự điều khiển lạ, chuỗi thoát terminal (ANSI escape sequences).
- Ngăn ngừa tiêm nhiễm mã nhắc lệnh (Prompt Injection) vào ngữ cảnh của tác tử AI.

## Đầu vào Bằng chứng Ứng dụng được Lưu giữ

Tác tử có thể tham chiếu trực tiếp đến các Bằng chứng đã được tạo ra trong các bước trước thông qua `evidence_id`:
- Giảm thiểu việc phải gửi lại toàn bộ dữ liệu tệp lớn giữa các lượt hội thoại.
- Giữ cho cửa sổ ngữ cảnh của mô hình ngôn ngữ luôn gọn gàng và hiệu quả.

## Danh tính và Khám phá (Identity and Discovery)

- **Tên máy chủ**: `rea` hoặc `morluto/rea`.
- **Giao thức**: Model Context Protocol (phiên bản tương thích chuẩn 2024-11-05 và mới hơn).
- **Khả năng (Capabilities)**: Hỗ trợ `tools`, `resources`, và `prompts`.

## Mục tiêu Ban đầu khi Khởi động

REA có thể được khởi động với một đường dẫn tệp nhị phân mục tiêu định sẵn qua cờ CLI hoặc biến môi trường:

```sh
rea serve-mcp --target ./application.exe
```

## Các phương thức Truyền tải (Transports)

- **`stdio`**: Giao tiếp chuẩn thông qua luồng tiêu chuẩn vào/ra (`stdin`/`stdout`). Đây là chế độ mặc định và tối ưu nhất cho Claude Desktop, VS Code, và các CLI agents.
- **`sse` (Server-Sent Events) / HTTP**: Phù hợp cho triển khai dịch vụ từ xa hoặc môi trường đám mây đa người dùng.

## Tiến độ và Hủy bỏ (Progress and Cancellation)

- Máy chủ hỗ trợ gửi thông báo tiến độ (`notifications/progress`) định kỳ trong các tác vụ dài.
- Tôn trọng yêu cầu hủy bỏ (`notifications/cancelled`) từ phía máy khách để giải phóng CPU và RAM ngay lập tức.

## Phạm vi Yêu cầu và Hiệu ứng Cục bộ

Mọi thao tác phân tích của REA đều là thao tác đọc hoặc chỉ ghi vào các thư mục tạm thời/bộ đệm được kiểm soát. REA không bao giờ tự ý sửa đổi tệp nhị phân gốc trừ khi được yêu cầu rõ ràng qua công cụ vá mã cụ thể.

## Dữ liệu Ghidra Tĩnh và Ánh xạ Nguồn

Khi làm việc với Ghidra:
- Địa chỉ bộ nhớ luôn được chuẩn hóa theo địa chỉ ảo tương đối (RVA) hoặc VA cơ sở nhất quán.
- Ánh xạ giữa dòng mã giả C và các lệnh hợp âm (assembly instructions) được bảo toàn.

## Cài đặt Phân tích có thể Chỉnh sửa

Tác tử có thể điều chỉnh cấu hình phân tích trong quá trình chạy:
- Độ sâu tìm kiếm của đồ thị gọi hàm (`max_depth`).
- Bật/tắt phân tích giải mã chuỗi tự động.

## Quảng bá Lược đồ (Schema Advertisement)

### Hồ sơ Lược đồ Thu gọn (Compact Input Schema Profile)

Khi bật `REA_COMPACT_SCHEMA=true`, REA rút gọn các mô tả tham số của công cụ thành phiên bản cô đọng nhất, giúp tiết kiệm tới 60% token định nghĩa công cụ trong prompt hệ thống.

### Phạm vi Quảng bá Họ Công cụ (Tool Family Advertising Scope)

Cho phép chọn lọc các nhóm công cụ cần kích hoạt:
- `native`: Các công cụ phân tích nhị phân mã máy (Ghidra, Hopper, PE/ELF/Mach-O).
- `managed`: Các công cụ phân tích .NET, Java, Unity.
- `web`: Các công cụ phân tích JavaScript, quan sát trình duyệt CDP, và bắt gói tin mạng.
- `artifact`: Các công cụ giải nén container, firmware, và đóng gói.

### Lược đồ Đầu ra (Output schemas)

Mọi phản hồi đều tuân theo hợp đồng JSON schema xác định trước, giúp tác tử dễ dàng phân tích cú pháp bằng code hoặc function calling.

## Danh mục Tự động Sinh (Generated Catalog)

Tài liệu tham khảo đầy đủ về từng công cụ và tham số được tự động sinh và cập nhật tại `docs/mcp-prompts.md`.
