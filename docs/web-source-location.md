# Vị trí nguồn website được thu thập (Captured Website Source Locations)

`trace_web_source_location` ánh xạ một vị trí đã chọn trong script được giữ lại bởi `export_web_scripts` thông qua một source map cục bộ được ghép đôi rõ ràng. Quy trình làm việc CLI/MCP dùng chung xác minh danh tính byte thực tế của manifest, script và map, đồng thời trả về vị trí gốc và nguồn được nhúng nội dòng. Nó không cần trình duyệt, không tìm nạp URL và không thực thi mã ứng dụng.

```bash
rea trace-web-source-location   /analysis/scripts/manifest.json 0   /analysis/app.js.map 'https://example.test/maps/app.js.map?v=7#context'   1 25 --json
```

Sáu đối số vị trí chọn manifest, chỉ mục script tính từ 0, đường dẫn map tuyệt đối, ngữ cảnh URL map tuyệt đối, dòng được tạo tính từ 1 và cột UTF-16 tính từ 0. Các đối số MCP là:

```json
{
  "manifest_path": "/analysis/scripts/manifest.json",
  "script_index": 0,
  "source_map": {
    "path": "/analysis/app.js.map",
    "url": "https://example.test/maps/app.js.map?v=7#context"
  },
  "generated_position": { "line": 1, "column": 25 }
}
```

URL cung cấp ngữ cảnh để giải quyết `sources` và `sourceRoot` bằng quy tắc phân tích cú pháp URL ECMA-426; nó không phải là lệnh tìm nạp. Một `sourceRoot` không rỗng làm tiền tố cho mỗi nguồn, chỉ thêm `/` khi root chưa kết thúc bằng dấu gạch chéo. Bản đồ của webpack 5 và tsc (`"sourceRoot": ""`) do đó giải quyết đối chiếu với URL bản đồ.

## Ý nghĩa của Bằng chứng

Tra cứu tuân theo vị trí được tạo lớn nhất nhỏ hơn hoặc bằng điểm đã chọn, qua các dòng, và trả về **mọi** phân đoạn tại vị trí đó. Nó bao gồm các ánh xạ chỉ được tạo (generated-only). Điều này tuân theo [ECMA-426 GetOriginalPositions](https://tc39.es/ecma426/#sec-getoriginalpositions), với các dòng tính từ 1 trong hợp đồng công khai của REA.

Mã băm nội dung nhúng xác định mã hóa UTF-8 của chuỗi `sourcesContent` đã giải mã, bao gồm cả phần kết thúc dòng của nó. Văn bản bản đồ thô được giữ lại trong `raw_result` của Bằng chứng, bao gồm các phần mở rộng không xác định.

## Giới hạn tài nguyên và adapter

Adapter sử dụng [`@jridgewell/trace-mapping` 0.3.31](https://github.com/jridgewell/trace-mapping) upstream đã khóa mà không sửa đổi mã của nó. Nó xác thực cấu trúc map/VLQ trước khi cấp phát, bình thường hóa các độ lệch được lập chỉ mục lồng nhau trước khi làm phẳng ngược dòng, kiểm tra ranh giới và chỉ mục section, và giữ lại danh tính khai báo thô.

Một tiến trình con Node thuộc sở hữu giải mã bản đồ. Thư mục yêu cầu riêng của nó bị xóa trước khi trả về thành công. Các cờ V8 rõ ràng đặt ngân sách 192 MiB cho old-generation và 8 MiB cho semi-space, một worker V8 và `decode_timeout_ms` (mặc định 20000).
