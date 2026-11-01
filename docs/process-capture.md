# Ghi lại Hoạt động Tiến trình (Process Capture)

REA cung cấp khả năng quan sát và ghi lại hành vi của tiến trình đang chạy trên hệ điều hành máy chủ, bao gồm các lời gọi hệ thống (syscalls), truy cập tệp tin, giao tiếp mạng và hoạt động tiến trình con.

## Hỗ trợ Máy chủ (Host Support)

- **Windows**: Sử dụng Event Tracing for Windows (ETW) hoặc API Procmon/Process Hacker để giám sát hoạt động tệp, registry và mạng.
- **Linux**: Sử dụng `ptrace`, `strace`, hoặc eBPF để thu thập các sự kiện hệ thống cấp thấp.
- **macOS**: Hỗ trợ giám sát thông qua DTrace hoặc Endpoint Security Framework.

## Ghi lại một Lệnh (Capture a command)

Khởi chạy một lệnh mới và ghi lại toàn bộ hoạt động của nó:

```sh
rea capture-process --cmd "npm run build" --output ./build-trace.json --json
```

Bản ghi thu thập bao gồm:
- **Cây tiến trình**: Tất cả các tiến trình con được tạo ra (`spawn`/`exec`), kèm theo PID, lệnh dòng lệnh đầy đủ và mã thoát.
- **Hoạt động I/O tệp**: Danh sách các tệp được mở, đọc, ghi hoặc xóa.
- **Kết nối mạng**: Các địa chỉ IP đích, cổng kết nối và giao thức được thiết lập trong suốt quá trình chạy.

## So sánh Hai Bản ghi Thu thập (Compare two captures)

So sánh hành vi giữa hai lần thực thi lệnh khác nhau:

```sh
rea compare-process-captures ./run1.json ./run2.json --output ./diff-trace.json --json
```

Báo cáo phân tích:
- Các tệp được truy cập trong lần chạy thứ hai nhưng không có trong lần chạy đầu.
- Sự thay đổi về số lượng và loại tiến trình con.
- Các yêu cầu mạng mới phát sinh.

## Băm Báo cáo Độc lập với Đầu ra Terminal

Đầu ra của terminal thường chứa các chuỗi ngẫu nhiên (dấu thời gian, thanh tiến trình, chuỗi màu ANSI). REA chuẩn hóa dữ liệu hành vi cốt lõi để tạo ra mã băm tất định (deterministic hash), cho phép kiểm tra tính lặp lại (reproducibility) của các quy trình dựng phần mềm.

## Chi tiết Kịch bản và Kết quả (Scenario and result details)

### Tiến độ (Progress)

Trong quá trình thu thập dài, REA phát ra các sự kiện tiến độ định kỳ:
- Số lượng sự kiện hệ thống đã thu thập được.
- Mức tiêu thụ CPU và bộ nhớ hiện tại của cây tiến trình.

### Kịch bản có Tương tác (Scenario with interactions)

REA hỗ trợ gửi tín hiệu hoặc đầu vào tự động (`stdin`) tới tiến trình mục tiêu tại các thời điểm định trước theo kịch bản khai báo.

### Khoảng thời gian Hoàn tất (Finalization interval)

Sau khi tiến trình chính kết thúc, REA chờ thêm một khoảng thời gian ngắn (mặc định 2-5 giây) để đảm bảo toàn bộ tiến trình con và các bộ đệm I/O hệ thống đã được xả sạch trước khi chốt bản ghi.
