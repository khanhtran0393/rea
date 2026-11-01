# Metadata phụ thuộc nhị phân Rust

`inspect_rust_binary` / `inspect-rust-binary` trả lời những tên crate, phiên bản, nguồn và cạnh phụ thuộc nào được nhúng bởi cargo-auditable trong một tệp thực thi hoặc thư viện cục bộ đã chọn. Lệnh đọc tệp mà không cần chạy nó và không yêu cầu cài đặt Rust, mục tiêu nhị phân đang hoạt động, Hopper, Ghidra hay IDA.

```bash
rea inspect-rust-binary /absolute/path/to/application --json
```

CLI giải quyết các đường dẫn tương đối so với thư mục làm việc của bên gọi. MCP yêu cầu một đường dẫn hệ thống tệp tuyệt đối:

```json
{
  "name": "inspect_rust_binary",
  "arguments": { "path": "/artifacts/application" }
}
```

## Các quan sát

Phân phối hoàn chỉnh vẫn là mặc định. Đối với câu hỏi về phụ thuộc, hãy chọn các bản ghi cần thiết ở lệnh gọi đầu tiên thay vì trả về toàn bộ đồ thị:

```json
{
  "name": "inspect_rust_binary",
  "arguments": {
    "path": "/artifacts/application",
    "view": {
      "collection": "packages",
      "names": ["serde", "serde_derive"],
      "fields": ["name", "version", "source", "dependencies"]
    }
  }
}
```

`detail: "summary"` trả về dải section, chỉ mục gốc, số lượng package/edge/cycle và số lượng mã băm trình biên dịch. Một `view` sẽ tự động chọn phân phối thu gọn. Bằng chứng cha hoàn chỉnh được giữ lại trong phiên MCP và có thể xuất khẩu. Các lệnh gọi `inspect_analysis_view` tiếp theo chấp nhận `parent_evidence_id` đó:

```json
{
  "source": {
    "kind": "retained-evidence",
    "evidence_id": "ev_<parent digest>"
  },
  "view": {
    "kind": "rust",
    "collection": "edges",
    "from_indices": [12],
    "limit": 20
  }
}
```

Bộ lọc gói chấp nhận `names`, `versions` chính xác và `indices` gốc; bộ lọc cạnh chấp nhận `indices`, `from_indices` và `to_indices` cạnh gốc. Các bộ lọc giao nhau và một mảng bộ lọc trống không chọn gì. `offset` đếm các bản ghi khớp; `limit` mặc định là 64.

CLI sử dụng các khung nhìn tương tự và yêu cầu một bundle đầu ra độc quyền cho phân phối thu gọn:

```bash
rea inspect-rust-binary /artifacts/application --json   --view '{"collection":"packages","names":["serde"],"fields":["name","version"]}'   --evidence_output /artifacts/rust-evidence.json
```

Tên section ELF được giải quyết thông qua bảng chuỗi tiêu đề section. Bảng tên nén Zlib và Zstandard được giải mã trong ngân sách cấu trúc native. Giải mã Zstandard cũng sử dụng giới hạn cửa sổ 128 MiB.

Khi có section `.dep-v0`, `auditable_metadata` chứa:
- `section_name`, `.dep-v0` cho ELF và PE hoặc `__DATA,.dep-v0` cho Mach-O;
- `file_offset` và `section_bytes`, dải tệp chính xác của section phụ thuộc trong artifact đã chọn;
- `compressed_bytes`, độ dài luồng zlib được sử dụng cho giới hạn tỷ lệ giải nén, và `uncompressed_bytes`.
