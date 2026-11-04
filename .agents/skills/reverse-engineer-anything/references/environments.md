# Môi trường tác tử từ một ứng dụng trực tiếp (Agent environments from a live application)

Sử dụng hướng dẫn này khi người dùng yêu cầu một môi trường tác tử (một gói AgentEnv bundle) của một ứng dụng web: một thư mục mà `agent-env` triển khai, có các công cụ MCP trả lời từ những gì một lần ghi nhận ứng dụng đã quan sát và phục vụ các thao tác ghi mà lần ghi nhận đó đã xác nhận. Quy trình làm việc sẽ ghi nhận ứng dụng một lần bằng `capture_browser_scenario`, kiểm tra xem môi trường sẽ phục vụ những gì, ghi nhận lại nếu gợi ý chỉ ra một khoảng trống, sau đó xuất gói bằng `export_environment_bundle`. Đối với tệp HAR mà người dùng đã có sẵn, hãy chuyển thẳng đến phần [xuất và bàn giao](#xuat-goi-va-ban-giao-export-and-hand-off) thông qua `inspect_web_network_capture`, hoặc sử dụng `rea environment-build <capture.har> <dir>`.

## Trước khi ghi lại (Before recording)

- Chỉ ghi nhận trên một phiên bản (instance) mà người dùng xác nhận là dùng một lần (disposable) hoặc được tạo dữ liệu mẫu cho mục đích này: mỗi bước ghi sẽ tạo, chỉnh sửa hoặc xóa dữ liệu của nó. Nếu người dùng chưa nói rõ, hãy hỏi trước bất kỳ kịch bản nào có thao tác ghi. REA không tự khởi động, tạo dữ liệu mẫu hoặc thiết lập lại (reset) ứng dụng.
- Thông tin xác thực lấy từ các biến môi trường do người dùng chỉ định. Khai báo mỗi biến dưới dạng một secret của kịch bản, `{secret_id, environment_variable}`; biến này phải được đặt trong môi trường mà server MCP của REA đã khởi động cùng. Tuyệt đối không bao giờ hỏi mật khẩu trực tiếp trong khung chat. Các secret đã khai báo sẽ được che giấu (redacted) khi bản ghi được lưu.
- Khảo sát ứng dụng trước tiên bằng `inspect_web_page` (và `capture_web_screenshot` khi bố cục giao diện quan trọng): bộ định vị (locators) của biểu mẫu đăng nhập, và các lệnh gọi API đằng sau các trang liệt kê và chỉnh sửa các thực thể mà người dùng yêu cầu.
- `inspect_web_page` không thể tự đăng nhập. Để xem các trang phía sau màn hình đăng nhập, hãy chạy một `capture_browser_scenario` ngắn để đăng nhập và mở chúng, với `"capture": {"after_each_step": ["accessibility"]}` (`"dom"` cho mã đánh dấu). Đọc cây của một bước bằng `inspect_analysis_view`, `view: {kind: "browser-scenario", collection: "text", step_index, artifact: "accessibility", offset: 0, limit: 2048}`, phân trang bằng `offset`. Tùy chọn thu thập chỉ bao gồm `after_each_step`, `at_end`, `events`, `network` và `limits`.

## Ghi lại một phiên để môi trường có thể học các thao tác ghi (Record a session an environment can learn writes from)

Một lần gọi `capture_browser_scenario` duy nhất sẽ ghi lại toàn bộ phiên. Bắt đầu từ cấu trúc sau, với bộ định vị, đường dẫn và tiêu đề của chính ứng dụng:

```json
{
  "browser": { "mode": "launch", "executable_path": "/usr/bin/chromium" },
  "start_url": { "url": "http://127.0.0.1:3000/login" },
  "secrets": [
    { "secret_id": "admin_password", "environment_variable": "APP_PASSWORD" }
  ],
  "capture": {
    "network": {
      "request_body": true,
      "response_body": true,
      "header_values": true
    }
  },
  "actions": [
    {
      "step_id": "user",
      "action": "fill",
      "timeout_ms": 15000,
      "locator": { "kind": "css", "selector": "input[name=username]" },
      "value": { "source": "literal", "value": "admin" }
    },
    {
      "step_id": "password",
      "action": "fill",
      "timeout_ms": 15000,
      "locator": { "kind": "css", "selector": "input[name=password]" },
      "value": { "source": "secret", "secret_id": "admin_password" }
    },
    {
      "step_id": "submit",
      "action": "click",
      "timeout_ms": 15000,
      "locator": { "kind": "css", "selector": "button[type=submit]" }
    }
  ]
}
```

- Sử dụng các bước `request` khi có sẵn các endpoint API đã biết để thực hiện các thao tác ghi và đọc lại nhanh chóng, không bị ảnh hưởng bởi thay đổi DOM.
- Đặt thời hạn `timeout_ms` cho từng bước để tránh bị treo vô hạn nếu một phần tử không xuất hiện.
- Các bước `request` không tự động đi theo chuyển hướng (redirects): một chuyển hướng 3xx sẽ làm bước đó thất bại, và `expected_status` không thể chỉ định mã 3xx. Hãy sử dụng đường dẫn mà chuyển hướng trỏ đến, chẳng hạn như đường dẫn có dấu gạch chéo ở cuối.
- Thiết lập `expected_status` trên mọi bước, để một lệnh gọi bị từ chối sẽ làm thất bại bước đó và các bước sau báo cáo là chưa chạy, thay vì ghi lại các lần đọc lại rỗng.

Giữ tổng số yêu cầu trong khoảng 60 đến 160 yêu cầu. Một kịch bản không có bước `request` sẽ được ghi lại chính xác như trước.

Bản ghi thu thập trả về toàn bộ Evidence của nó, bao gồm mọi phần thân đã ghi, thường nhiều hơn mức một client hiển thị nội tuyến. Trước khi kiểm tra độ bao phủ, hãy xác nhận rằng mọi mục của `normalized_result.steps` đều có `status: "completed"`; nếu client đã lưu kết quả vào một tệp, hãy tìm kiếm `"status":"failed"` trong đó và đọc `error` của bước đó. Một bước đăng nhập hoặc ghi thất bại sẽ hủy bỏ tất cả các bước phía sau nó. Sử dụng `evidence_id` của kết quả cho việc kiểm tra độ bao phủ và xuất gói; không dán ngược Evidence vào một công cụ khác.

## Kiểm tra độ bao phủ (Check coverage)

Sử dụng `inspect_analysis_view` với `source: {kind: "retained-evidence", evidence_id}` (ID của bản ghi thu thập) và `view: {kind: "environment-coverage", template_rules: ["after-uid-literal", "opaque-token", "sibling-variance", "prefix-variance"]}`. Nó báo cáo các công cụ đọc được trả lời từ trạng thái hoặc được phát lại (`reads.recorded_reasons`), các công cụ ghi được phục vụ, và từng công cụ ghi bị giữ lại cùng với `reason` và `hint` của nó. Nó không áp dụng các kiểm tra danh mục và kích thước trạng thái tại thời điểm phục vụ; thao tác xuất sẽ báo cáo những điều đó.

Ghi nhận lại một thao tác ghi bị giữ lại tối đa hai lần, và chỉ khi có khoảng trống ghi nhận. Dừng lại ngay lập tức khi lý do xuất phát từ hành vi ứng dụng, và báo cáo thao tác ghi đó bị giữ lại kèm gợi ý của nó:

| Lý do (`Reason`) | Việc cần làm |
| --- | --- |
| `no-state-target` | Kiểm tra mẫu đường dẫn và các chế độ xem đã ghi; ghi lại các lần đọc bị thiếu. Dạng tuyến đường có thể không được hỗ trợ làm mục tiêu biến đổi. |
| `no-observation` | Ghi nhận lại: đọc lại thực thể theo id sau thao tác ghi; đọc nó trước tiên đối với thao tác cập nhật hoặc xóa. |
| `unconfirmed` | Ghi nhận lại: đọc thực thể và danh sách của nó sau thao tác ghi. |
| `incomplete-entities` | Ghi nhận lại: đọc thực thể thông qua mọi chế độ xem liệt kê nó. |
| `unexplained-change` | Ghi nhận lại một lần với lệnh gọi thứ hai, khác biệt của cùng một thao tác ghi. |
| `no-call` | Ghi nhận lại một lần với một thao tác ghi thành công. |
| `side-effect` | Dừng lại: ứng dụng thay đổi các thực thể khác. |
| `contradicted` | Dừng lại: một lần đọc sau đó không khớp với hiệu ứng được mô hình hóa. |
| `body-unusable` | Ghi nhận lại một thao tác ghi UI một lần với các bước `request` (phần thân của nó không bị lộ); nếu không thì dừng lại. |

Sử dụng tên tạm (scratch names) mới cho mỗi lần thử, và xóa những gì lần thử trước đã tạo trước khi ghi lại (thông qua giao diện người dùng, hoặc bước `request` với id mà bản ghi thu thập hiển thị): môi trường bắt đầu từ những gì bản ghi đọc trước tiên, do đó một đối tượng còn sót lại sẽ trở thành dữ liệu mẫu ban đầu mà các tác vụ ghi sau đó sẽ tìm thấy. Các bản ghi thu thập kịch bản trình duyệt mới bao gồm thời gian yêu cầu, do đó một bản ghi có đầy đủ thời gian có thể học đồng hồ ghi và phục vụ các bản cập nhật đóng dấu thời gian sửa đổi. Các bản ghi cũ hơn hoặc thiếu phần bắt đầu yêu cầu sẽ giữ nguyên thứ tự bản ghi và không học đồng hồ. Khi độ bao phủ báo cáo giới hạn đó, hãy ghi lại bằng phiên bản hiện tại trước khi kết luận rằng thay đổi dấu thời gian không thể được phục vụ.

## Xuất gói và bàn giao (Export and hand off)

Sử dụng `export_environment_bundle` với `source: {kind: "retained-evidence", evidence_id}` (hoặc `{kind: "evidence-file", path}`), một `output_directory` tuyệt đối chưa tồn tại nhưng thư mục cha của nó đã tồn tại, và một `name` (bắt buộc khi ứng dụng chạy trên `localhost` hoặc địa chỉ IP). Lệnh này trả về độ bao phủ một lần nữa, `tool_count`, các tệp đã ghi và các cảnh báo; `dry_run: true` sẽ báo cáo mà không ghi tệp. Gói này cài đặt `rea-agents` theo phiên bản của REA này từ sổ đăng ký; khi người dùng chạy một bản dựng REA chưa phát hành, hãy truyền `package_tarball` mà họ cung cấp (tệp tarball `rea-agents` đã đóng gói) thay thế.

Thông báo cho người dùng biết có bao nhiêu công cụ trả lời từ trạng thái, thao tác ghi nào được phục vụ, thao tác nào bị giữ lại và lý do tại sao, cùng lệnh chạy (`agent-env run <folder>`). Cho biết rằng gói này chứa các phản hồi đã ghi và đường dẫn cục bộ của bản ghi (đối với kịch bản trình duyệt, là đường dẫn của tệp thực thi trình duyệt), và liệt kê mọi con trỏ cảnh báo `credential-named-members` để xem xét. Tuyệt đối không bao giờ tự ý xuất bản hoặc chia sẻ gói này.

Xem [môi trường tác tử](https://github.com/morluto/rea/blob/main/docs/agent-environments.md) để biết những gì trạng thái, phát lại và gói xác lập, và [quy trình bằng chứng](evidence-workflows.md) cho các chế độ xem manifest và trạng thái.
