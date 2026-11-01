# Mối quan hệ module website được thu thập (Captured Website Module Relationships)

`trace_web_module_imports` / `rea trace-web-module-imports` truy vết các lệnh import và re-export native ES gửi đi của một script từ một [bản thu thập đã xuất](website-script-export.md). Lệnh này xác minh SHA-256 và số lượng byte của nguồn đã chọn, bảo toàn danh tính URL trình duyệt, và trả về nội dòng mọi ứng viên thu thập khớp. Nó không tải lại các tài nguyên website.

## Chọn nguồn và ngữ cảnh

Cung cấp tệp thực thi Chromium do bên gọi cung cấp có hỗ trợ `import.meta.resolve`. REA không cài đặt hay nâng cấp trình duyệt. Việc xác minh thực tế bao gồm Linux x64; các máy chủ khác chưa nhận được xác minh quy trình làm việc thực tế của tính năng này.

```bash
REA_BROWSER_EXECUTABLE=/absolute/path/to/chrome   rea trace-web-module-imports /analysis/export/manifest.json 0   --importer-url 'https://example.test/app/main.js?build=2#entry'   --import-map-path /analysis/import-map.json   --import-map-base-url https://example.test/maps/   --json
```

Đầu vào MCP thể hiện các lựa chọn tương tự:

```json
{
  "manifest_path": "/analysis/export/manifest.json",
  "script_index": 0,
  "importer_url": "https://example.test/app/main.js?build=2#entry",
  "import_map": {
    "path": "/analysis/import-map.json",
    "base_url": "https://example.test/maps/"
  }
}
```

`script_index` được tính từ số 0 trong `manifest.scripts`. Nếu bỏ qua `importer_url`, dấu vết sẽ chọn URL nguồn được báo cáo. Không thể thiết lập base tài liệu của script nội dòng chỉ từ URL đó; hãy chọn ngữ cảnh rõ ràng khi biết trước. `import_map` là tùy chọn và chỉ đọc JSON cục bộ. Các byte chính xác và URL cơ sở của nó được giữ lại; việc chọn nó không chứng minh map đã được cài đặt trong trang được thu thập. Các ngữ cảnh importer HTTP(S) không có thông tin người dùng URL được hỗ trợ ban đầu. URL được báo cáo không thể sử dụng sẽ khiến ngữ cảnh importer không khả dụng; hãy cung cấp `importer_url` rõ ràng khi biết giá trị của nó. URL sai định dạng được cung cấp rõ ràng sẽ được báo cáo theo trường đầu vào đó. Các URL sử dụng các scheme không được hỗ trợ vẫn là các giới hạn năng lực.

## Diễn giải bằng chứng (Evidence)

- Các lệnh import tĩnh, re-export và import động theo chuỗi ký tự (literal dynamic imports) nhận URL đã giải quyết của engine native hoặc thông báo từ chối chính xác. Chuỗi rỗng được giữ lại.
- Các import động dạng tính toán (computed dynamic imports) giữ lại biểu thức và vị trí nguồn của chúng với độ phân giải unknown dạng `computed-specifier`. Các import/export chỉ dành cho kiểu dữ liệu đã bị xóa (erased type-only) và ID bundler được suy diễn bị loại trừ khỏi các xác nhận import native.
- Vị trí sử dụng độ lệch và cột UTF-16, với dòng tính từ 1. Chẩn đoán khôi phục phân tích cú pháp và đầu vào không thể phân tích được hiển thị rõ ràng.
- Các biến thể truy vấn (query) và đoạn (fragment) vẫn tách biệt. URL được báo cáo chính xác là một ứng viên `exact-reported-url`. URL phản hồi bị loại bỏ đoạn là một ứng viên nội dung `response-url-without-fragment` yếu hơn. Nhiều phiên bản được thu thập đều được trả về; quá trình trace không chọn phiên bản nào làm chuẩn (canonical).
- Metadata ứng viên vẫn là bằng chứng thu thập được báo cáo; chỉ có các byte của nguồn đã chọn mới được xác minh lại. Các mục tiêu bị thiếu/tải lười (lazy) có thể giải quyết thành công mà không có ứng viên nào được thu thập. Mọi mối quan hệ đều báo cáo việc thực thi là unknown.
- Các khai báo source-map vẫn là khai báo; thao tác này không đọc hay tạo byte map, ánh xạ nguồn gốc, quyền sở hữu worker hay tính nhân quả UI.

Nguồn đã chọn được nạp từ `files/` dưới thư mục hiện tại của manifest. Việc di chuyển vị trí lưu giữ riêng các đường dẫn được báo cáo ban đầu của manifest và đường dẫn nguồn hiện tại thực tế. Đường dẫn không an toàn, thư mục nguồn symlink, tệp không thông thường, mã băm/kích thước sai và UTF-8 lỗi sẽ thất bại rõ ràng. Byte nguồn bị lỗi hoặc đường dẫn nguồn không an toàn do manifest khai báo là các lỗi artifact. Tệp manifest hoặc import-map do bên gọi chọn bị lỗi sẽ xác định trường đầu vào được cung cấp.

## Engine và vòng đời

Một ngữ cảnh trình duyệt tạm thời được sở hữu chạy stub giải quyết đáng tin cậy của REA. Ứng dụng đã chọn được phân tích cú pháp dưới dạng dữ liệu và không bao giờ được thực thi bởi trace này. Các yêu cầu trang được đáp ứng bằng tài liệu/stub tổng hợp của REA hoặc bị chặn; không có tài nguyên website nào được tải xuống. Điều này không thiết lập hộp cát (sandbox) hệ điều hành hay ngăn chặn mạng trên toàn trình duyệt. Phiên bản engine và báo cáo native được giữ lại trong Bằng chứng.

Các nguồn không có import ký tự không yêu cầu trình duyệt. Manifest, nguồn và import map có các giới hạn đầu vào lần lượt là 32 MiB, 16 MiB và 4 MiB. Độ phân giải native có hạn chót 20 giây; việc dọn dẹp có thể kéo dài hạn chót đó. Một renderer được yêu cầu và không có tuyên bố giới hạn bộ nhớ máy chủ tổng hợp. Thao tác hủy chỉ đóng trình duyệt được sở hữu; dọn dẹp thất bại không thể trả về một trace thành công. `REA_BROWSER_NO_SANDBOX=true` truyền tùy chọn khởi chạy tương ứng của Chromium khi máy chủ yêu cầu; nó không kích hoạt ranh giới bảo mật của REA.

Xem [kiểm thử](testing.md) để biết oracle nạp native và luồng thực tế CLI/MCP.

Tên tệp nguồn cục bộ đã chọn cung cấp ngữ pháp phân tích cú pháp. Tệp JavaScript giữ lại các import động native ngay cả trong các biểu thức quan hệ; tệp TypeScript sử dụng ngữ pháp TypeScript dùng chung và bỏ qua các biểu thức import-type bị xóa khỏi việc trace module runtime. Việc import nguồn lịch sử giữ lại các phụ thuộc kiểu đó một cách riêng biệt. Các quan sát của trình phân tích cú pháp và việc giải quyết URL native không chứng minh việc thực thi.
