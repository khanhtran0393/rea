# Kiểm kê tĩnh tuyến đường JavaScript (Route Inventory)

`analyze_javascript_application` và `analyze_web_bundle` giữ lại `route_inventory` dùng chung bên cạnh đồ thị và các quan sát endpoint hiện có của chúng. Bản kiểm kê trả lời các mẫu tuyến đường (route patterns) mà mã nguồn gợi ý, từng mẫu bắt nguồn từ đâu, và những phần nào vẫn chưa xác định. Lệnh không bao giờ tự gửi yêu cầu tuyến đường, thực thi script, mở rộng sitemap hoặc xác lập khả năng tiếp cận.

Chọn khung nhìn thu gọn trong lệnh gọi phân tích MCP:

```json
{
  "detail": "summary",
  "view": {
    "kind": "javascript-routes",
    "collection": "routes",
    "where": { "path_prefix": "/reports" },
    "fields": ["path", "classification", "confidence"],
    "limit": 20
  }
}
```

Đối với CLI, truyền cùng lựa chọn dưới dạng `--view '<JSON>' --detail summary --evidence_output routes.json`. Bundle Bằng chứng hoàn chỉnh được ghi mà không ghi đè lên các tệp hiện có.

Sử dụng `normalized_result.parent_evidence_id` được trả về với `inspect_analysis_view` trên cùng kết nối. Các view không mở lại tệp nguồn hay kết nối lại với trình duyệt.

## Phân trang các quan sát và suy diễn endpoint đã thu thập

Đối với `analyze_web_bundle`, chọn `detail: "summary"` và `view: {kind: "web-bundle", collection: "endpoints", limit: 32}`. Truy vấn parent được trả về bằng `inspect_analysis_view` để đọc `endpoints`, `inferences`, hoặc `summary` mà không cần thu thập lại trang hoặc lặp lại phân tích JavaScript.

Cả hai collection hàng đều chấp nhận `indices`, `values`, `script_keys`, `fields`, `offset`, và `limit` chính xác; endpoints cũng chấp nhận `mechanisms`, và inferences chấp nhận `kinds`. Mỗi hàng giữ lại `source_index` của nó trong collection ban đầu.

Các collection bao gồm `summary`, `routes`, `occurrences`, `sources`, và `unknowns`. ID mẫu liên kết với đường dẫn chính xác. ID nguồn liên kết với khóa nguồn, đường dẫn, URL ban đầu, mã băm nội dung, mã băm container và bất kỳ ngữ cảnh áp dụng nguồn Next nào được giữ lại. Các lần xuất hiện nhận diện cơ chế bộ dò tại một vị trí nguồn.
