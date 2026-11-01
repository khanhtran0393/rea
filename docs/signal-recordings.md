# Giải mã bản ghi tín hiệu (Signal Recording Decoding)

`decode_signal_recording` trả lời những gì một bộ giải mã giao thức đã chọn quan sát được trong một bản ghi hiện có. Nó sử dụng **sigrok-cli 0.7.2** do bên gọi cung cấp cùng với libsigrokdecode trên **Linux**. `prlimit` của util-linux là bắt buộc (mặc định `/usr/bin/prlimit`, có thể ghi đè bằng đường dẫn tuyệt đối `REA_SIGROK_PRLIMIT_COMMAND`). Bộ giới hạn đặt giới hạn không gian địa chỉ mềm và cứng trước khi bộ giải mã bắt đầu. macOS và Windows bị từ chối vì adapter này không thể thực thi giới hạn đó trên các máy chủ đó. Lệnh không tự thu thập mẫu hay giao tiếp với phần cứng.

```sh
export REA_SIGROK_COMMAND=/absolute/path/to/sigrok-cli
rea decode-signal-recording /artifacts/uart.vcd   --decoders uart:rx=RX:baudrate=9600 --input_format vcd   --annotations uart=rx-data --format json
```

Yêu cầu MCP tương ứng là:

```json
{
  "name": "decode_signal_recording",
  "arguments": {
    "path": "/artifacts/uart.vcd",
    "decoders": "uart:rx=RX:baudrate=9600",
    "input_format": "vcd",
    "annotations": "uart=rx-data"
  }
}
```

`decoders` sử dụng ngăn xếp bộ giải mã của sigrok cùng cú pháp tùy chọn/kênh. Tên kênh phải khớp với bản ghi; REA không đoán cách đấu dây, tốc độ baud hoặc giao thức. Mỗi kênh giải mã đã chọn cần một mục tiêu, ví dụ `uart:rx=RX`.

Việc bỏ qua `input_format` yêu cầu tự động phát hiện của sigrok. REA sử dụng một bản sao riêng tư bất biến có tên tệp giữ lại phần mở rộng cuối cùng của bản ghi đã chọn. Việc bỏ qua `annotations` sẽ giữ lại mọi lớp chú thích.

Đầu ra mặc định là 50 dòng phân tách bằng ký tự xuống dòng (LF). Sử dụng `line_offset` và `line_limit` cho các trang tiếp theo, hoặc `line_limit: null` trong MCP / `--all_lines` trong CLI cho tất cả đầu ra còn lại. `output_window` báo cáo `total_lines`, `returned_lines` và `has_more`; các giá trị này đếm các dòng hiển thị, **không phải các chú thích hay mẫu**.

Mỗi bộ chọn bị giới hạn ở 8.192 đơn vị mã UTF-16 và tổng các đối số bộ chọn được mã hóa là 32 KiB trước khi khởi chạy.

Bằng chứng trả về bao gồm đường dẫn/mã băm đầu vào, các lựa chọn chính xác, đường dẫn/mã băm công cụ đã cấu hình và giải quyết, biểu ngữ phiên bản đầy đủ, văn bản chú thích với dải số mẫu, và stderr.
