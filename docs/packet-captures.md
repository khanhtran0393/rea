# Kiểm tra bản thu thập gói tin đã ghi lại (Recorded Packet Capture)

`inspect_packet_capture` trả lời những trường giao thức đã chọn nào mà TShark quan sát thấy trong một bản thu thập hiện có. Các bản ghi Network, USB hoặc Bluetooth có thể sử dụng cùng giao diện trường/bộ lọc khi TShark đã cài đặt hỗ trợ đóng gói và bộ mổ xẻ (dissectors) của chúng. Công cụ này không thu thập lưu lượng, mở giao diện hay điều khiển thiết bị.

Cung cấp bản cài đặt TShark 4.x hiện có **trên Linux**, với `prlimit` của util-linux tại `/usr/bin/prlimit` (hoặc ghi đè bằng `REA_TSHARK_PRLIMIT_COMMAND`). Các máy chủ không được hỗ trợ, bao gồm macOS và Windows, bị từ chối trước khi kiểm tra vì tích hợp này yêu cầu giới hạn không gian địa chỉ được thực thi.

```sh
export REA_TSHARK_COMMAND=/absolute/path/to/tshark
rea inspect-packet-capture /artifacts/device.pcapng   --fields ip.addr --fields udp.port --fields data.data   --display_filter 'udp' --format json
```

Lệnh gọi MCP tương ứng:

```json
{
  "name": "inspect_packet_capture",
  "arguments": {
    "path": "/artifacts/device.pcapng",
    "fields": ["ip.addr", "udp.port", "data.data"],
    "display_filter": "udp"
  }
}
```

`frame.number` luôn được bao gồm. Các giá trị lặp lại là các mảng chuỗi có thứ tự, ví dụ `"ip.addr": ["192.0.2.1", "192.0.2.2"]`. Các trường bị thiếu vẫn vắng mặt; chúng không được chuyển đổi thành số 0 hoặc mảng trống.

Các bộ chọn bị giới hạn trước khi mở rộng argv: tối đa 1.024 trường và 1.024 quy tắc `decode_as`, 8.192 đơn vị mã UTF-16 cho mỗi bộ chọn, và tổng cộng 32 KiB đối số bộ chọn được mã hóa.

Đầu ra được phân trang sau khi lọc: `match_limit` mặc định là 50 và `match_offset` là 0. Kết quả giữ lại số frame ban đầu và báo cáo `output_window` với số lượng khớp trong phạm vi đọc đầu vào, số lượng trả về và `has_more`.

Các lựa chọn bổ sung:
- `display_filter`: cú pháp bộ lọc hiển thị Wireshark.
- `decode_as`: các ghi đè bộ mổ xẻ rõ ràng, chẳng hạn như `["udp.port==5555,dns"]`.
- `packet_limit`: số lượng gói tin đầu vào tối đa được đọc trước khi lọc.
- `two_pass`: mặc định là true cho các trường cần các gói sau và phụ thuộc lắp ráp lại.
