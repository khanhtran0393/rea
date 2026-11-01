# Lộ trình phát triển REA (REA Roadmap)

[Trình theo dõi nền tảng](https://github.com/khanhtran0393/rea/issues/32), [trình theo dõi tiến trình/Hopper](https://github.com/khanhtran0393/rea/issues/48), và [trình theo dõi trình duyệt](https://github.com/khanhtran0393/rea/issues/39) theo dõi năng lực và công tác xác minh. Các công cụ và thiết lập có sẵn được ghi chép trong [hướng dẫn điều tra](https://rea.tools/guides/), [hướng dẫn cài đặt](installation.md), và [danh mục được tạo tự động](mcp-contracts.md#generated-catalog).

## Các ưu tiên (Priorities)

- Mở rộng xác minh kiến trúc native, kiểu dữ liệu và lệnh gọi gián tiếp (indirect-call).
- Kết nối các phát hiện tĩnh và quan sát thời gian chạy trên các tầng ứng dụng.
- Cải thiện so sánh mã được quản lý (managed-code) bị xáo trộn và các liên kết native đã xác minh.
- Mở rộng độ bao phủ tiến trình, giao thức, hệ thống tệp, kết nối lại và so sánh bản dựng.
- Đánh giá các provider và mục tiêu bổ sung khi quy trình làm việc của chuyên viên phân tích chứng minh có khoảng trống năng lực.

## Tiêu chí tiếp nhận (Admission Criteria)

Năng lực mới phải trả về Bằng chứng (Evidence) hữu ích thông qua quy trình làm việc CLI/MCP dùng chung, bảo toàn các quan sát và điểm chưa xác định, đồng thời khai báo thẩm quyền thực tế và các ràng buộc vòng đời. Các tác tử kết hợp các thử nghiệm bằng các lệnh, script và fixture server thông thường; việc điều phối tùy chỉnh cần có yêu cầu được chứng minh.

Các tuyên bố về provider và nền tảng yêu cầu [luồng xác minh chuỗi công cụ thực tế](testing.md#real-toolchain-verification-lanes) tương ứng.
Xem [thiết kế công cụ](tool-design.md) để chọn ranh giới công cụ và [ngữ nghĩa Ghidra](ghidra-provider.md) cho hợp đồng provider chuyên sâu hiện có.

Thiết lập vẫn mang tính bổ sung, tái sử dụng các công cụ hiện có và yêu cầu phê duyệt đối với các thay đổi cài đặt Hopper và cấu hình được tiết lộ. Các lựa chọn trình cài đặt bổ sung cần một chuỗi công cụ được triển khai và phạm vi rõ ràng; quá trình thiết lập không tự cài đặt hoặc nâng cấp các runtime hoặc các phần phụ thuộc không liên quan.
