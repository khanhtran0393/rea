# Thiết bị đo Frida (Frida Instrumentation)

REA hiển thị khả năng khám phá thiết bị và tiến trình Frida, các phiên đính kèm (attach) hoặc tạo mới (spawn), và công cụ đo JavaScript do người dùng tạo thông qua CLI và MCP. Các liên kết Node là một phụ thuộc gói tùy chọn và chỉ nạp khi thao tác Frida chạy. REA không cài đặt Frida server trên mục tiêu.

Đối với thiết bị từ xa, truyền host với cổng tùy chọn (`host[:port]`; đặt địa chỉ IPv6 trong dấu ngoặc vuông) và bất kỳ tùy chọn token, chứng chỉ, origin hoặc keepalive nào trên thao tác đó. Khoảng thời gian keepalive được đo bằng giây, khớp với Frida Core 17.23.2. Các giá trị xác thực chỉ được giữ lại cho lệnh gọi hoặc phiên đang hoạt động và bị loại trừ khỏi Bằng chứng (Evidence). Không nhúng thông tin xác thực vào địa chỉ endpoint.

MCP cung cấp các công cụ `list_frida_devices`, `list_frida_processes`, `start_frida_session`, `load_frida_script`, `resume_frida_session`, `unload_frida_script`, `frida_session_status`, và `close_frida_session`. Một tiến trình được spawn bắt đầu ở trạng thái tạm dừng để script có thể nạp trước. Thao tác đóng sẽ hủy nạp script phiên và gỡ đính kèm (detach). Nếu một spawn vẫn tạm dừng, REA chỉ tiếp tục nó sau khi việc dọn dẹp script hoặc gỡ phiên xác nhận thiết bị đo đã được giải phóng; nếu cả hai đường dọn dẹp đều thất bại, mục tiêu vẫn tạm dừng và lỗi dọn dẹp được giữ lại. REA không bao giờ hủy (kill) mục tiêu. Các phiên MCP kết thúc cùng tiến trình server. Lệnh CLI một lần `instrument-with-frida` chấp nhận một script nội dòng hoặc tệp script, quan sát thông báo trong một khoảng thời gian có giới hạn và trả về Bằng chứng cùng mã băm nguồn và các thông báo thu thập được.

`instrument-with-frida --json` trả về bản ghi Bằng chứng trực tiếp, khớp với MCP. Người dùng trình bao bọc `{evidence, cleanup_error}` CLI trước đây nên đọc bản ghi tại gốc JSON và quan sát dọn dẹp phiên tại `normalized_result.cleanup_error`. Nếu việc dọn dẹp provider cuối cùng thất bại hoặc việc hủy bỏ xảy ra trong quá trình đó, lỗi có kiểu giữ lại Bằng chứng đã hoàn thành trong `details.partial_observation`. Các lệnh khám phá giữ lại báo cáo thiết bị hoặc tiến trình của chúng trong cùng một trường.

Để kiểm tra trạng thái và độ bao phủ thu thập, truyền `{"session_id":"SESSION_ID","detail":"summary"}` tới `frida_session_status`. Mỗi lệnh gọi đọc trạng thái phiên quan sát được và giữ lại quan sát hoàn chỉnh. Phản hồi là tóm tắt Bằng chứng `inspect_analysis_view` với danh tính phiên, trạng thái, script, `messages_retained`, và `messages_truncated`; tải trọng thông báo vẫn nằm trong `parent_evidence_id` của nó. Số lượng bao gồm các thông báo trong hàng đợi được giữ lại, không phải mọi sự kiện mục tiêu. Bỏ qua `detail` hoặc đặt `detail=complete` giữ nguyên phản hồi đầy đủ hiện có.

Trích đoạn các trường đã chọn trong bản tóm tắt:

```json
{
  "summary": {
    "state": "running",
    "messages_retained": 50,
    "messages_truncated": false
  }
}
```

Sử dụng `get_evidence_bundle` với `detail="summary"` và `filters={"evidence_id":"PARENT_ID"}` để khám phá metadata cha mà không cần tải trọng thông báo. Bộ lọc chỉ áp dụng cho phân phối tóm tắt. Sử dụng `export_evidence_bundle` cho bundle chuẩn hoàn chỉnh, sau đó chọn bản ghi có `evidence_id` khớp với `normalized_result.parent_evidence_id` từ tóm tắt trạng thái. Để sử dụng `inspect-analysis-view` CLI hiện có, trích xuất bản ghi đó thành tệp Evidence JSON độc lập và chọn nó làm nguồn tệp Evidence với `view={"kind":"summary"}`. Việc đọc Bằng chứng được lưu giữ không làm mới một phiên đang hoạt động hoặc xác minh tiến trình sau khi thoát hoặc tái sử dụng PID.

Dọn dẹp khởi động thất bại sẽ giữ lại các handle Frida đã lấy, bao gồm một spawn tạm dừng chưa đính kèm, cho các lần thử dọn dẹp sau. Kết nối từ xa của nó vẫn được giữ cho đến khi việc gỡ phiên và tiếp tục mục tiêu hoàn tất. Các lệnh CLI đợi dọn dẹp provider trước khi trả về; MCP giữ lại các tài nguyên này để dọn dẹp khi server đóng. Việc tắt server sẽ ngừng tiếp nhận các thao tác Frida mới và đợi công việc khởi động, khám phá và hàng đợi đã được tiếp nhận hoàn tất trước khi giải phóng tài nguyên. Một dọn dẹp thất bại vẫn được báo cáo và có thể thử lại bằng cách đóng server lần nữa, thay vì coi một phiên chưa đăng ký là đã giải phóng hoàn toàn.

Script Frida thực thi bên trong mục tiêu đã chọn với bộ nhớ, hệ thống tệp và quyền mạng của tiến trình đó. Tra cứu thiết bị cục bộ đợi 1000 ms. Bản phát hành này không có ghi đè runtime cho lần tra cứu đó. Endpoint từ xa không sử dụng tra cứu đó. Thu thập thông báo bị giới hạn ở 16 MiB và báo cáo việc cắt ngắn. Giới hạn các trường metadata, kiểu và văn bản, cùng giới hạn node JSON cũng là cố định. Nguồn script không được lưu trữ trong Bằng chứng; đường dẫn đã chọn và mã băm SHA-256 của nó thì có. Đối với một tệp, `source_sha256` xác định các byte đã chọn ban đầu. UTF-8 không hợp lệ tiếp tục sử dụng giải mã thay thế; Bằng chứng sau đó ghi lại giới hạn đó và `raw_result.source_representation`. Bằng chứng giữ lại lựa chọn mục tiêu phi xác thực như thiết bị hoặc endpoint, chế độ, chương trình hoặc PID, và đối số spawn. Thông tin xác thực luôn bị loại trừ.

Tài liệu [Frida Core API](https://frida.re/docs/frida-core-api/) mô tả quản lý thiết bị, phiên và nạp script; [JavaScript API](https://frida.re/docs/javascript-api/) mô tả thiết bị đo phía mục tiêu. [Luma](https://luma.frida.re/) là GUI độc lập của Frida; REA không tự động hóa hoặc điều khiển nó. `npm run verify:frida` yêu cầu máy chủ Linux có liên kết Frida tùy chọn và sử dụng một fixture Node.js cục bộ ngắn hạn do bộ xác minh tạo ra. Thêm `-- --address HOST:PORT --pid PID` để chạy luồng đính kèm từ xa riêng biệt đối với mục tiêu do bên gọi chọn.
