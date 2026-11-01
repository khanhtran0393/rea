# Cấu hình Khách và Xử lý Sự cố (Client Configuration and Troubleshooting)

Tài liệu này cung cấp hướng dẫn tích hợp máy chủ MCP REA vào các ứng dụng khách, tác tử và môi trường phát triển phổ biến. REA tương thích với chuẩn giao thức Model Context Protocol (MCP) trên cả hai hình thức truyền tải: `stdio` (tiến trình cục bộ) và `sse` / HTTP streaming.

## Trình khởi chạy và PATH (Launchers and PATH)

Đảm bảo rằng lệnh `rea` hoặc tệp nhị phân Node.js có thể truy cập được từ biến môi trường `PATH` của ứng dụng khách. Đối với môi trường desktop trên macOS hoặc Windows, các biến môi trường của ứng dụng GUI có thể khác với shell terminal; hãy sử dụng đường dẫn tuyệt đối nếu cần.

## Cài đặt thời gian chạy và Thời hạn kiểm tra của máy khách

Phân tích nhị phân và dịch ngược có thể mất thời gian đối với các tệp lớn. Khuyến nghị định cấu hình thời gian chờ yêu cầu (deadline) trong cấu hình khách ít nhất là:
- Yêu cầu thông thường: 30-60 giây.
- Yêu cầu dịch ngược Ghidra/JADX ban đầu: 180-300 giây.

## Tên công cụ và Khám phá phía Máy khách

Một số khách MCP giới hạn số lượng công cụ hiển thị. REA cho phép lọc họ công cụ thông qua cờ khởi chạy hoặc cấu hình biến môi trường:
- `REA_TOOL_FAMILIES=native,dotnet,artifact`
- `REA_COMPACT_SCHEMA=true`: Bật cấu hình lược đồ thu gọn để tiết kiệm ngữ cảnh cho mô hình ngôn ngữ.

## Fiddler Everywhere cùng với REA

REA hỗ trợ phân tích trực tiếp các tệp lưu trữ `.saz` của Fiddler Everywhere. Không cần khởi chạy đồng thời tiến trình Fiddler nếu bạn đã lưu phiên mạng.

## Cấu hình cho từng Ứng dụng Khách

### VS Code (Cline / Roo Code / Continue)

Thêm cấu hình vào tệp `cline_mcp_settings.json` hoặc cấu hình MCP tương ứng:

```json
{
  "mcpServers": {
    "rea": {
      "command": "npx",
      "args": ["-y", "@morluto/rea", "serve-mcp"]
    }
  }
}
```

### GitHub Copilot CLI

Định cấu hình trong phần mở rộng công cụ của GitHub Copilot để tự động gọi các công cụ REA khi phân tích dự án.

### Claude Desktop

Thêm cấu hình vào tệp `claude_desktop_config.json`:
- **macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Windows**: `%APPDATA%\Claude\claude_desktop_config.json`

```json
{
  "mcpServers": {
    "rea": {
      "command": "npx",
      "args": ["-y", "@morluto/rea", "serve-mcp"]
    }
  }
}
```

### Claude Code

Khởi động phiên với công cụ mở rộng:

```sh
claude --mcp "npx -y @morluto/rea serve-mcp"
```

### Kimi Code

Tích hợp qua tệp cấu hình tác tử với kết nối stdio trỏ tới lệnh `rea serve-mcp`.

### Pi

Hỗ trợ kết nối MCP tiêu chuẩn. Lưu ý giới hạn hiển thị mã nguồn dòng lệnh trên Pi để tránh tràn bộ đệm.

### Qwen Code

Định cấu hình máy chủ MCP trong tệp thiết lập mô hình của Qwen Code.

### Antigravity

Trong Google Antigravity, REA được tích hợp trực tiếp dưới dạng skill hoặc máy chủ MCP tùy chỉnh trong thư mục cấu hình `.gemini/config/`.

### Gemini CLI

Sử dụng cờ `--mcp` để truyền thông số kết nối tới tiến trình `rea serve-mcp`.

### Codex và Devin

Cung cấp cấu hình môi trường với quyền thực thi công cụ cục bộ cho tiến trình REA.

#### Codex với mô hình Moonshot hoặc Kimi

Sử dụng chế độ `--compact-schema` để tối ưu hóa độ dài của prompt định nghĩa công cụ.

### OMP, Hermes, Qoder CLI, Factory Droid, Trae, WorkBuddy, ZCode, FreeBuff

Đều hỗ trợ kết nối qua lệnh thực thi chuẩn:

```json
{
  "command": "rea",
  "args": ["serve-mcp"]
}
```

### DeepSeek Harness

Tối ưu hóa độ dài ngữ cảnh bằng cách giới hạn các họ công cụ được quảng bá (`--tool-families`).

## Các cài đặt máy khách khác

- `REA_LOG_LEVEL`: Đặt thành `debug`, `info`, `warn`, hoặc `error`.
- `REA_CACHE_DIR`: Tùy chỉnh thư mục lưu bộ đệm giải mã và ảnh chụp phân tích.

## Lựa chọn và Bảo trì Skill

Khi cập nhật REA, hãy chạy `rea update-skills` để làm mới các tệp hướng dẫn skill trong thư mục cấu hình của tác tử.
