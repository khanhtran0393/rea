# Hiện vật ứng dụng JavaScript và Electron (JavaScript and Electron application artifacts)

Đối với JavaScript được thu thập hoặc đóng gói cần khôi phục mã nguồn có thể đọc được, hãy sử dụng `recover_javascript_sources` với `path` tuyệt đối và để trống `output_directory`. Cung cấp các tệp runtime/shared/lazy liên quan trong `additional_paths` khi được quảng bá, sau đó cùng phân tích thư mục `analysis_input` được trả về. Mỗi đầu vào giữ lại bản chụp (snapshot), mã băm (digest) và nguồn gốc phạm vi byte của riêng nó; các đầu vào và mã được khôi phục không bao giờ được thực thi. Quá trình khôi phục yêu cầu Wakaru do bên gọi cung cấp trên Linux x64; xem [hướng dẫn khôi phục](https://github.com/morluto/rea/blob/main/docs/javascript-recovery.md).

Sử dụng `analyze_javascript_application` trực tiếp trên gói ASAR hoặc cây thư mục đã giải nén do người vận hành cung cấp. Khi câu hỏi liên quan đến các đường dẫn mã nguồn đã biết và lược đồ được kết nối có quảng bá `roots`, hãy cung cấp các đường dẫn tương đối của chúng trong `roots: ["src/auth", "src/billing"]` để tập trung đồ thị ứng dụng vào các hệ thống con đó. Đồ thị được trả về bảo toàn các đường dẫn chưa giải quyết được và các mối quan hệ đa vai trò; nó không lọc theo mô-đun hoặc mở rộng ra ngoài cây thư mục đã cung cấp.

Khi phân tích một gói web đã thu thập hoặc tệp JavaScript độc lập, hãy sử dụng `analyze_web_bundle`. Công cụ này xây dựng một đồ thị ứng dụng tương thích từ các tệp JavaScript, source map và tài nguyên liên quan được cung cấp, mà không yêu cầu cấu trúc gói ASAR hoặc cây thư mục dự án hoàn chỉnh.

Đối với các ứng dụng Electron, hãy bắt đầu bằng việc kiểm tra các cầu chì (fuses) đóng gói bằng `inspect_electron_fuses` để xác định các cờ bảo mật và khả năng gỡ lỗi được cấu hình tại thời điểm biên dịch, chẳng hạn như `EnableNodeCliInspectArguments` hoặc `RunAsNode`.

## Kiểm kê các ứng viên định tuyến mà không trả về đồ thị ứng dụng (Inventory route candidates without returning the application graph)

Khi lược đồ được quảng bá hỗ trợ `view`, hãy sử dụng `detail: "summary"` với `view: {kind: "javascript-routes", collection: "routes", fields: ["path", "confidence"], limit: 20}`.
Chế độ xem tương tự cũng hoạt động với `analyze_web_bundle` trên các nguồn được thu thập đã chọn trước đó.
Evidence nguồn hoàn chỉnh được lưu giữ; hãy sử dụng `parent_evidence_id` của nó với `inspect_analysis_view` để phân trang các trang tiếp theo hoặc chọn các trường bổ sung như `handler_node_id`, `methods` hoặc `middleware`. Lược đồ báo cáo các ứng viên định tuyến được phát hiện tĩnh; các tuyến đường được tạo động tại thời điểm runtime vẫn là chưa xác định.

## Đọc một mô-đun mà không lặp lại phân tích (Read a module without repeating analysis)

Sử dụng ID cha thực tế được trả về, trên cùng một kết nối, làm `source.evidence_id`. Các ID sau đây là phần giữ chỗ để thay thế bằng các giá trị được trả về thực tế. Khi chưa biết node ID của mô-đun, hãy phân trang các định danh trước khi chọn một mục. Các trang chỉ trả về node ID, đường dẫn và vai trò; hãy chọn kích thước trang hữu ích và theo dõi `coverage.next_offset` khi cần một trang khác:

```json
{
  "source": {
    "kind": "retained-evidence",
    "evidence_id": "ev_0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef"
  },
  "view": {
    "kind": "javascript-modules",
    "collection": "identities",
    "offset": 0,
    "limit": 50
  }
}
```

Khi node ID của mô-đun đã được xác định, hãy yêu cầu chế độ xem chi tiết của mô-đun đó để kiểm tra các lệnh import, export, chuỗi ký tự, các lệnh gọi API được quan sát và mối quan hệ chứa đựng:

```json
{
  "source": {
    "kind": "retained-evidence",
    "evidence_id": "ev_0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef"
  },
  "view": {
    "kind": "javascript-modules",
    "collection": "item",
    "node_id": "mod_abcdef0123456789"
  }
}
```

Thao tác này tránh việc truyền tải lại toàn bộ đồ thị ứng dụng chỉ để đọc một tệp duy nhất.

## Chỉ lần vết câu hỏi còn lại (Trace only the remaining question)

Sử dụng `trace_application_feature` trên Evidence ứng dụng cha cho một node ID, tuyến đường, chuỗi ký tự, API, kênh IPC, mô-đun hoặc export native cụ thể. Chọn `incoming` cho các mối quan hệ dẫn đến seed, `outgoing` cho các mối quan hệ rời khỏi nó, và `both` khi câu hỏi cần cả hai chiều. Việc duyệt lần vết bao gồm các mối quan hệ chứa đựng (containment relationships), do đó ngay cả một seed hẹp cũng có thể mở rộng các tệp chứa nó; hãy chọn hướng tiếp cận phù hợp nhất với câu hỏi điều tra.

```json
{
  "application": {
    "kind": "retained-evidence",
    "evidence_id": "ev_0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef"
  },
  "seed": {
    "kind": "ipc_channel",
    "value": "app:request-license"
  },
  "direction": "incoming"
}
```

## Khôi phục khi hiển thị quá khổ (Recover an oversized display)

Một phản hồi quá khổ sẽ báo cáo `resource_constraint` và `details.resource: "transport"`. Tái sử dụng `details.reported_limits.evidence_reference` với `inspect_analysis_view` để lấy bản tóm tắt, một mô-đun đơn lẻ, hoặc một trang định danh mô-đun ổn định. Lệnh `export_evidence_bundle` ghi toàn bộ phiên chuẩn vào một tệp do bên gọi chọn. Các quy trình này giữ nguyên Evidence ban đầu và ngăn chặn các vòng lặp thất bại do truyền tải lại cùng một khối dữ liệu quá lớn qua khung MCP.

Đối với đầu ra CLI, dữ liệu JSON hoàn chỉnh được truyền luồng ra stdout mà không bị giới hạn kích thước khung đơn lẻ của MCP. Bạn có thể chuyển hướng đầu ra vào tệp bằng cách sử dụng `--evidence_output /path/to/evidence.json` để kiểm tra ngoại tuyến hoặc phân tích bằng các công cụ khác.

## Tái sử dụng Evidence ứng dụng (Reusing application Evidence)

Khi được server kết nối quảng bá, các công cụ lần vết và so sánh ứng dụng chấp nhận Evidence nội tuyến hoàn chỉnh hoặc:

```json
{
  "kind": "retained-evidence",
  "evidence_id": "ev_<64 ký tự hex viết thường>"
}
```

cho đầu vào ứng dụng của chúng (`application`, hoặc `left`/`right`). Lệnh `inspect_analysis_view` sử dụng cùng một định dạng tham chiếu lưu giữ trong `source`, hoặc Evidence nội tuyến khả chuyển, để chiếu các chế độ xem mục tiêu cụ thể mà không cần chạy lại bộ phân tích.

Khi so sánh hai phiên bản ứng dụng bằng `compare_applications`, hãy cung cấp cả hai đối tượng ứng dụng dưới dạng tham chiếu lưu giữ để nhận bản phân tích khác biệt có cấu trúc về các tuyến đường, mô-đun, kênh IPC và API đã thêm, sửa đổi hoặc loại bỏ.
