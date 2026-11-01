# Tóm tắt bản thu thập gói tin (Packet Capture Summaries)

REA tóm tắt các tệp pcap/pcapng đã lưu thông qua `tshark` và `capinfos` của Wireshark 4.x do bên gọi chọn. Sử dụng `inspect_packet_capture` cho các trường, bộ lọc và quy tắc decode-as đã chọn; sử dụng họ công cụ này cho kiểm kê địa chỉ, DNS, SNI, protocol-stack và URI. Lệnh không tự thu thập lưu lượng truy cập hoặc tìm nạp các mục tiêu đã ghi lại.

## Cung cấp công cụ

```sh
export REA_TSHARK_COMMAND=/absolute/path/tshark
export REA_CAPINFOS_COMMAND=/absolute/path/capinfos
# Chỉ Linux; mặc định /usr/bin/prlimit
export REA_TSHARK_PRLIMIT_COMMAND=/absolute/path/prlimit
rea inspect-pcap-client
rea inspect-pcap-capture /captures/session.pcapng --json
```

Nếu không có lệnh công cụ rõ ràng, họ công cụ này giải quyết chúng thông qua PATH và ghi lại nguồn lựa chọn đó. Trên Linux và macOS, nó chạy công cụ được tìm thấy trong mục PATH tuyệt đối đầu tiên chứa tệp thực thi. Việc kiểm tra capture yêu cầu Linux với util-linux prlimit hoặc Windows với Job Objects thuộc sở hữu.

## Chỉ kiểm tra các dữ kiện cần thiết

MCP `inspect_pcap_capture` chấp nhận `path` và tùy chọn `detail: "summary"`. Phân phối hoàn chỉnh vẫn là mặc định. Bản tóm tắt cam kết và giữ lại cùng Bằng chứng hoàn chỉnh, trả về danh tính, metadata capture, số lượng quan sát được, số lượng được giữ lại, độ bao phủ nguồn và độ dài chẩn đoán mà không có danh sách lớn. Sau đó sử dụng `normalized_result.parent_evidence_id` của nó với kiểu nguồn `retained-evidence` của `inspect_analysis_view`:

```json
{"kind":"pcap","facet":"endpoints","offset":0,"limit":20}
{"kind":"pcap","facet":"dns-queries","offset":0,"limit":20}
{"kind":"pcap","facet":"dns-addresses","query":"example.com","offset":0,"limit":20}
{"kind":"pcap","facet":"tls-sni","offset":0,"limit":20}
{"kind":"pcap","facet":"diagnostics","stream":"tshark","offset":0,"limit":4096}
```

Các facet hàng khác là `protocols`, `http-uris` và `unattributed-addresses`. Mặc định là offset 0 và limit 64.

## Diễn giải nguồn gốc và độ bao phủ

`target.sha256` liên kết các quan sát với một snapshot capture ổn định đã tiếp nhận. Cả hai công cụ đều đọc bản sao đó, bản sao này được băm lại sau khi phân tích.

`capture_start` và `capture_end` là các thời điểm UTC RFC 3339 được dẫn xuất từ giây kỷ nguyên `capinfos -S`, giữ lại tất cả các chữ số thập phân của trình tạo (lên đến nano giây).

Địa chỉ được đếm một lần cho mỗi gói, bao gồm cả loopback và các lần xuất hiện IP lồng nhau. Phân bổ địa chỉ DNS yêu cầu một câu hỏi trong một phản hồi và số lượng section và RR owner/type đầy đủ, nhất quán. Chỉ các bản ghi A/AAAA thuộc section trả lời khớp với câu hỏi đó mới được liên kết. TLS SNI là metadata bắt tay được ghi lại, không phải danh tính tiến trình hay bằng chứng về tính xác thực của đích đến.
