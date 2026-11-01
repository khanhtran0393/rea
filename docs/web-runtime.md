# Quy kết nguồn runtime website (Website Runtime Source Attribution)

REA tích hợp khả năng kiểm tra Chrome DevTools Protocol (CDP) native và độ bao phủ V8 chính xác thông qua hai công cụ có thể kết hợp. Cung cấp một endpoint CDP loopback hiện có và một mục tiêu trang từ `list_browser_targets`. Nếu bỏ qua origins, REA chọn origin HTTP(S) hiện tại của trang. Trình duyệt và trang vẫn thuộc quyền sở hữu bên ngoài.

## Quan sát việc thực thi

```bash
rea observe-web-execution http://127.0.0.1:9222 TARGET_ID   --observation-ms 10000 --json
```

MCP: `observe_web_execution` với `cdp_endpoint`, `target_id`, `allowed_origins` tùy chọn và `observation_ms` tùy chọn (mặc định 10000).

Việc bắt đầu độ bao phủ chính xác sẽ đặt lại các bộ đếm và ngăn chặn việc thực thi tối ưu hóa cho đến khi REA dừng nó. Điều này làm thay đổi hành vi định thời/JIT. REA yêu cầu số lượng lệnh gọi chính xác và các dải chi tiết, lấy một mẫu đặt lại, dừng độ bao phủ, vô hiệu hóa các miền đã kích hoạt và gỡ bỏ kết nối của chính nó. Nó không điều hướng, nhấp chuột, đánh giá mã ứng dụng đã chọn hoặc đóng trang.

Kết quả giữ lại:
- Mục tiêu đã chọn, danh tính frame và document loader; dấu thời gian kích hoạt/kết thúc cục bộ và dấu thời gian bao phủ đơn điệu phụ trợ riêng biệt.
- Mọi dải hàm/khối của trình tạo được tiếp nhận, độ lệch kết thúc độc quyền và số lần thực thi theo đơn vị mã UTF-16, với các ranh giới đã xác minh khi văn bản chính xác có sẵn.
- Kiểm kê script của tài liệu chính, ID script phiên chính xác, tọa độ bắt đầu tài nguyên, khai báo sourceURL/source-map, mã băm của trình tạo và mã băm văn bản UTF-8 được tính toán độc lập.
- Các yêu cầu trong cửa sổ kích hoạt từ frame chính đã chọn và các origin đã chọn, các đối tượng khởi tạo được báo cáo đầy đủ, các điểm gọi (callsites) dẫn xuất và các liên kết ID script chính xác.

## Kiểm tra trình lắng nghe của một node (Listeners)

```bash
rea inspect-web-event-listeners http://127.0.0.1:9222 TARGET_ID '#submit' --json
```

MCP: `inspect_web_event_listeners` với cùng phạm vi và một CSS `selector` rõ ràng. Lựa chọn DOM native kiểm tra node khớp đầu tiên trong tài liệu chính.

Kết quả chứa các kiểu/cờ trình lắng nghe được đăng ký trực tiếp, tọa độ callback, danh tính/văn bản/mã băm nguồn và các ẩn số thực thi. REA sử dụng các thao tác DOM/DOMDebugger native và giải phóng nhóm đối tượng từ xa của chính nó. Nó không bao giờ tự gọi trình lắng nghe hay gửi sự kiện.
