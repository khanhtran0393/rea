# Artifact WebAssembly ngoại tuyến (Offline WebAssembly)

`inspect_wasm_artifact` / `inspect-wasm-artifact` kiểm tra các byte WASM cục bộ được chọn rõ ràng bằng cách sử dụng WABT 1.0.42 do bên gọi cung cấp. Không yêu cầu mục tiêu nhị phân đang hoạt động. REA không cài đặt công cụ, không tìm nạp artifact và không thực thi WASM.

`open_binary` không phải là đường dẫn kiểm tra WASM. Tiêu đề WASM v1 được nhận diện sẽ trả về `target_unavailable` với danh mục `unsupported_target` và ràng buộc `wasm_requires_static_inspection`. Hãy sử dụng công cụ không cần mục tiêu đó hoặc `rea inspect-wasm-artifact <path> --format json` với chuỗi công cụ bên dưới.

Đặt `REA_WABT_BIN_DIRECTORY` thành thư mục tuyệt đối chứa các tệp thực thi `wasm-validate`, `wasm-objdump` và `wasm2wat` (kèm `.exe` trên Windows). Tất cả ba công cụ phải báo cáo 1.0.42 trên stdout với stderr trống.

```bash
REA_WABT_BIN_DIRECTORY=/absolute/wabt/bin rea inspect-wasm-artifact   /artifacts/module.wasm --glue_paths /artifacts/glue.js   --candidate_paths /artifacts/other/module.wasm --format json
```

Đầu vào MCP tương đương:

```json
{
  "name": "inspect_wasm_artifact",
  "arguments": {
    "path": "/artifacts/module.wasm",
    "glue_paths": ["/artifacts/glue.js"],
    "candidate_paths": ["/artifacts/other/module.wasm"]
  }
}
```

Mỗi tệp thông thường được chọn được đọc thông qua trình đọc artifact ổn định hiện có; các liên kết mềm, tệp đang thay đổi và tệp trên 8 MiB sẽ thất bại đóng. Các byte WASM được sao chép vào một workspace riêng tư dưới dạng `module.wasm` chỉ đọc; mã băm của nó được kiểm tra lại trước và sau mỗi công cụ. WABT xác thực bằng `wasm-validate --enable-all --ignore-custom-section-errors`, báo cáo các section thông qua `wasm-objdump -h -x`, và giải mã thông qua `wasm2wat --enable-all --ignore-custom-section-errors`. Kết quả giữ lại SHA-256 và kích thước của byte đã chọn, dải tải trọng section, đầu ra tiêu đề/chi tiết đầy đủ, các dạng WAT import/export chính xác, WAT đã giải mã và SHA-256 UTF-8 của nó.
