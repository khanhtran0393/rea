# Giao diện Dòng lệnh (CLI) và Bằng chứng (Evidence)

Sử dụng CLI cho việc kiểm tra trực tiếp hoặc trong tập lệnh tự động hóa. Nó sử dụng cùng một quy trình ứng dụng và hợp đồng bằng chứng như máy chủ MCP của REA. Mỗi lần gọi CLI là một tiến trình riêng biệt; lưu kết quả khi bạn muốn sử dụng lại trong lệnh sau. Để biết quy trình làm việc cố định, không tương tác với các báo cáo được lưu giữ, hãy xem [phân tích tĩnh trong GitHub Actions](ci-analysis.md).

## Chạy một lệnh

Sau khi [cài đặt REA](installation.md), hãy sử dụng lệnh `rea`. Bạn cũng có thể chạy lệnh mà không cần cài đặt toàn cục:

```sh
# Chạy toàn cục sau khi cài đặt
rea --help

# Chạy trực tiếp qua npx
npx rea --help
```

## Đầu ra bản ghi JSON Lines

Thêm cờ `--json` hoặc `--jsonl` để xuất kết quả theo định dạng JSON có thể xử lý bằng máy:

```sh
rea inspect-artifact ./target.exe --json
```

Đầu ra này có thể dễ dàng chuyển tiếp sang các công cụ như `jq`:

```sh
rea inspect-artifact ./target.exe --json | jq .architecture
```

## Tự động hoàn thành Shell (Shell completion)

Kích hoạt gợi ý lệnh trên Bash, Zsh hoặc PowerShell:

```sh
# Cho Bash
eval "$(rea completion bash)"

# Cho Zsh
eval "$(rea completion zsh)"

# Cho PowerShell
rea completion powershell | Out-String | Invoke-Expression
```

## Cụm mã nguồn quản lý .NET (Managed assemblies)

Kiểm tra cấu trúc tệp nhị phân .NET PE, metadata, kiểu dữ liệu, phương thức và cờ làm mờ:

```sh
rea inspect-assembly ./App.dll --json
rea inspect-assembly ./App.exe --include-members --json
```

## Ảnh chụp nhanh SQLite (SQLite snapshots)

Kiểm tra cấu trúc và nội dung cơ sở dữ liệu SQLite:

```sh
rea inspect-sqlite ./database.db --json
rea inspect-sqlite ./database.db --table users --limit 50 --json
```

## Phân tích Nhị phân Native

Khám phá hàm, chuỗi ký tự, import/export và biểu tượng của tệp thực thi native:

```sh
rea inspect-native ./binary --functions --json
rea inspect-native ./binary --strings --json
```

## Lựa chọn nhà cung cấp phân tích (Choose a provider)

REA hỗ trợ nhiều nhà cung cấp phân tích mã máy native:
- `--provider ghidra`: Sử dụng công cụ Ghidra headless chuyên sâu.
- `--provider ida`: Tích hợp với trình phân tích IDA Pro.
- `--provider internal`: Phân tích tĩnh nội bộ tốc độ cao không cần cài đặt ngoài.

```sh
rea inspect-native ./binary --provider ghidra --decompile-main --json
```

## Lưu và sử dụng lại ảnh chụp phân tích (Save and reuse snapshots)

Lưu kết quả phân tích tốn kém vào tệp để truy vấn lại tức thì:

```sh
rea inspect-native ./large-binary --output-snapshot ./binary.rea-snap
rea query-snapshot ./binary.rea-snap --function main --json
```

## Nhập, xuất và so sánh Bằng chứng (Evidence)

```sh
# Xuất bằng chứng
rea export-evidence ./analysis-result.json --output ./evidence-bundle.zip

# So sánh hai kết quả phân tích
rea diff-evidence ./build1-evidence.json ./build2-evidence.json --json
```

## Nhập mã nguồn lịch sử (Import historical source)

Liên kết mã nguồn gốc hoặc các phiên bản trước với tệp nhị phân hiện tại:

```sh
rea correlate-source ./binary --source-dir ./v1.0-src/ --json
```

## Ghi lại hành vi thời gian chạy (Capture runtime behavior)

Ghi lại hoạt động của tiến trình hoặc kết nối mạng:

```sh
rea capture-process --cmd "./my-app --worker" --output ./process-trace.json
```

## Phục vụ MCP qua Streamable HTTP

Chạy REA như một máy chủ dịch vụ MCP hỗ trợ HTTP streaming:

```sh
rea serve-mcp --port 3000 --transport sse
```

## Phục vụ và xây dựng gói môi trường (Serve & Build Environment)

```sh
# Xây dựng môi trường từ tệp ghi mạng
rea environment-build ./traffic.har --output ./env-bundle --json

# Phục vụ môi trường cho tác tử
rea environment-serve ./env-bundle/manifest.json --port 8080
```

## Xuất và so sánh môi trường

```sh
rea environment-export ./env-bundle --archive ./env.tar.gz
rea environment-compare ./env-bundle --live-target https://api.example.com --json
```

## Báo cáo phiên bản và thời gian chạy

```sh
rea --version
rea doctor
```

Lệnh `rea doctor` kiểm tra sự hiện diện của Java, Ghidra, IDA Pro, JADX, trình duyệt Chromium, và báo cáo các công cụ khả dụng.

## Mã thoát và Trạng thái đầu ra (Output and exit status)

- `0`: Thực thi thành công, kết quả hợp lệ.
- `1`: Lỗi người dùng (tham số không hợp lệ, tệp không tìm thấy).
- `2`: Lỗi công cụ phụ trợ (Java/Ghidra thất bại hoặc quá thời gian chờ).
- `3`: Lỗi xác thực hoặc không đáp ứng ràng buộc an toàn.

### `inspect-devicetree`

Kiểm tra cây thiết bị phần cứng nhúng:

```sh
rea inspect-devicetree ./board.dtb --json
```

### PE TLS và Cấu hình Nạp (PE TLS and Load Configuration)

Kiểm tra Thread Local Storage và cấu hình tải bảo mật (`GuardCF`, `ControlFlowGuard`, `SEHandler`) trên Windows PE:

```sh
rea inspect-pe-load-config ./driver.sys --json
```
