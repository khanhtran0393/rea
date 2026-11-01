# Provider IDA

REA điều chỉnh một máy chủ IDA MCP hiện có thành các công cụ phân tích native của nó. Các profile IDA Pro hiện có sử dụng [mrexodia/ida-pro-mcp](https://github.com/mrexodia/ida-pro-mcp); một cầu nối native tùy chọn cho IDA Pro và IDA Free cũng được hỗ trợ. Cài đặt và cấu hình engine đã chọn cùng MCP upstream bằng hướng dẫn của họ. REA không tự cài đặt, nâng cấp, kích hoạt hay cấp phép cho chúng.

Chọn một chế độ vòng đời trong đăng ký JSON MCP upstream. Đặt `REA_IDA_MCP_CONFIG` thành tệp đó, sau đó chọn `ida` thông qua `REA_ANALYSIS_PROVIDER`, `--provider ida` của CLI, hoặc `open_binary.provider_id` của MCP.

| Chế độ   | Profile upstream    | Đã biết hoạt động                                      |
| -------- | ------------------- | ------------------------------------------------------ |
| attached | legacy GUI tools    | 1.4.0                                                  |
| headless | database supervisor | các bản dựng phục vụ `idb_open`, `idb_list`, `idb_close`|

## Phân tích một mục tiêu GUI đã mở sẵn

Profile đính kèm hỗ trợ **các công cụ legacy 1.4.0** upstream, bao gồm `get_metadata`, `list_functions`, và `decompile_function`. Sao chép `command`, `args`, và `env` tùy chọn của đăng ký upstream vào một tệp JSON:

```json
{
  "command": "python",
  "args": [
    "/absolute/path/to/ida_pro_mcp/server.py",
    "--ida-rpc",
    "http://127.0.0.1:13337"
  ]
}
```

```bash
export REA_IDA_MCP_CONFIG=/absolute/path/to/ida-mcp.json
export REA_ANALYSIS_PROVIDER=ida
rea function /absolute/path/to/program main --json
rea decompile /absolute/path/to/program main --json
```

Mở **tệp nhị phân đầu vào gốc** trong REA, không phải cơ sở dữ liệu `.idb` hoặc `.i64`. Mã băm SHA-256 của nó phải khớp với mã băm đầu vào được ghi lại của IDA. REA kiểm tra danh tính đính kèm và image base xung quanh mỗi yêu cầu phân tích.
