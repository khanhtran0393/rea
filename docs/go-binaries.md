# Metadata bản dựng nhị phân Go (Go Binary Build Metadata)

`inspect_go_binary` / `inspect-go-binary` trả lời phiên bản trình biên dịch Go, các module, các thay thế (replacements) và thiết lập bản dựng nào được nhúng trong một tệp thực thi hoặc thư viện cục bộ đã chọn. Lệnh đọc tệp mà không cần chạy nó và không yêu cầu cài đặt Go, mục tiêu nhị phân đang hoạt động, Hopper, Ghidra hay IDA.

```bash
rea inspect-go-binary /absolute/path/to/application --json
```

CLI giải quyết các đường dẫn tương đối so với thư mục làm việc của bên gọi. MCP yêu cầu đường dẫn hệ thống tệp tuyệt đối:

```json
{
  "name": "inspect_go_binary",
  "arguments": { "path": "/artifacts/application" }
}
```

Đối với danh tính và số lượng bản dựng, hãy chọn `detail: "summary"`. MCP giữ lại cùng Bằng chứng hoàn chỉnh và trả về `normalized_result.parent_evidence_id`. Đọc các phiên bản phụ thuộc đã chọn mà không cần mở lại tệp:

```json
{
  "name": "inspect_analysis_view",
  "arguments": {
    "source": {
      "kind": "retained-evidence",
      "evidence_id": "ev_<parent digest>"
    },
    "view": {
      "kind": "go",
      "collection": "dependencies",
      "paths": ["golang.org/x/crypto"],
      "fields": ["path", "version", "replacement"],
      "offset": 0,
      "limit": 10
    }
  }
}
```

Khung nhìn Go hỗ trợ `dependencies` (`paths` chính xác) và `settings` (`keys` chính xác), `indices` gốc tùy chọn, lựa chọn trường và phân trang.

Tóm tắt CLI một lần lưu Bằng chứng hoàn chỉnh trước khi trả về phép chiếu:

```bash
rea inspect-go-binary ./application --detail summary --evidence_output ./go-evidence.json --json
```

## Các quan sát

Kết quả Bằng chứng nội dòng chứa đường dẫn artifact, SHA-256, độ dài byte, định dạng container, kiến trúc, kích thước từ và thứ tự byte. Một bản ghi `build_info` được nhận diện chứa:
- `go_version`, văn bản UTF-8 nhúng;
- `go_version_bytes_base64`, bảo toàn các byte chuỗi trình biên dịch chính xác;
- độ lệch tệp và độ dài byte chính xác cho tiêu đề, chuỗi trình biên dịch và chuỗi module;
- `module_text`, phần thân thông tin module đã giải mã;
- `module.path`, đường dẫn package nhúng;
- `module.main`, đường dẫn module chính, phiên bản, checksum và thay thế tùy chọn;
- `module.dependencies`, cùng các trường và thứ tự ban đầu của chúng;
- `module.settings`, các mục khóa/giá trị có thứ tự và các khóa trùng lặp;
- `module.complete`, cho biết liệu mọi dòng văn bản module hiện diện có được phân tích cú pháp hay không.
