# Quan sát thụ động runtime Node và Electron

REA có thể đính kèm vào mục tiêu V8 Inspector của Node.js hoặc Electron đang chạy và giữ lại metadata về việc nạp script cùng ngữ cảnh thực thi. Các công cụ MCP `list_javascript_runtime_targets` và `observe_javascript_runtime` có các lệnh CLI tương đương là `rea list-javascript-runtime-targets` và `rea observe-javascript-runtime`. Các lệnh gọi thành công trả về Bằng chứng tất định.

Mỗi yêu cầu cung cấp endpoint Inspector loopback và ID mục tiêu để quan sát. Bề mặt này tách biệt khỏi CDP của trình duyệt, kiểm tra trang tệp Electron và Process Capture:

```bash
rea list-javascript-runtime-targets http://127.0.0.1:9229   --json
rea observe-javascript-runtime http://127.0.0.1:9229 TARGET_ID   --runtime-kind node --json
```

## Khởi động mục tiêu Node do người dùng sở hữu

Chạy ứng dụng đã chọn với Inspector được liên kết với loopback:

```bash
node --inspect=127.0.0.1:9229 ./app.mjs
```

Giữ nó chạy trong khi liệt kê các mục tiêu và quan sát ID đã chọn. Sử dụng một cổng rảnh và cùng một endpoint nguyên văn trong cả hai lệnh gọi REA. Dừng ứng dụng thông qua vòng đời bình thường của nó khi hoàn tất; REA ngắt kết nối nhưng không sở hữu hoặc chấm dứt nó.

Sử dụng `--inspect` cho quy trình thụ động này. `--inspect-brk` tạm dừng trước khi ứng dụng bắt đầu, và REA không bao giờ tiếp tục nó hoặc gửi `Runtime.runIfWaitingForDebugger`.

Đối với ứng dụng Electron đã đóng gói, hãy đọc [`inspect_electron_fuses`](electron-fuses.md) trước khi bắt đầu mục tiêu `electron-main`. Khi `EnableNodeCliInspectArguments` bị tắt hoặc bị xóa, Electron bỏ qua `--inspect` và `--inspect-brk`.

Trường `attached` trong kết quả mục tiêu runtime JavaScript và trình duyệt có thể nhận giá trị null: `true` và `false` chỉ được báo cáo khi quá trình khám phá cung cấp một giá trị boolean; `null` có nghĩa là endpoint đã bỏ qua hoặc không thể báo cáo trạng thái.

`list_javascript_runtime_targets` trả về mọi mục tiêu khả dụng trong một mảng nội dòng.

## Ranh giới giao thức thụ động

Provider chỉ gửi đúng hai lệnh giao thức:
- `Runtime.enable`
- `Debugger.enable`
