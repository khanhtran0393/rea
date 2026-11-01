# Trạng thái cầu chì Electron (Electron Fuse State)

`inspect_electron_fuses` / `inspect-electron-fuses` trả lời câu hỏi liệu một ứng dụng Electron đã đóng gói có chấp nhận `--inspect` và `--inspect-brk`, `ELECTRON_RUN_AS_NODE`, xác thực tính toàn vẹn ASAR được nhúng và chỉ tải mã ứng dụng từ `app.asar` hay không. Nó đọc luồng cầu chì (fuse wire) được lưu trữ trong tệp nhị phân Electron. Nó không khởi chạy ứng dụng, không sửa đổi byte cầu chì hoặc sử dụng mạng.

```bash
rea inspect-electron-fuses /Applications/Example.app --json
```

CLI phân giải đường dẫn tương đối dựa trên thư mục làm việc của bên gọi. MCP yêu cầu đường dẫn hệ thống tệp tuyệt đối và bảo toàn lựa chọn chính xác đó:

```json
{
  "name": "inspect_electron_fuses",
  "arguments": { "path": "/Applications/Example.app" }
}
```

Trên máy chủ POSIX, CLI bảo toàn các thành phần đường dẫn để các liên kết tượng trưng được lần theo trước các bước cha (`..`). Các lựa chọn gói (bundle) có thể kết thúc bằng `.` hoặc các bước cha qua các thư mục thực; một bước cha cuối cùng thông qua một liên kết tượng trưng thư mục sẽ bị từ chối. Đường dẫn Windows CLI giữ nguyên độ phân giải ổ đĩa và dấu phân cách native.

Hãy kiểm tra điều này trước khi dùng `observe_javascript_runtime` với `electron-main`. Khi `EnableNodeCliInspectArguments` bị tắt hoặc bị xóa, Electron bỏ qua các cờ Inspector, và danh sách mục tiêu sau đó có thể chỉ đơn giản là không hiển thị mục tiêu tiến trình chính nào.

## Những gì được đọc (What is Read)

Trên macOS, truyền gói `.app` hoặc tệp nhị phân Electron Framework. Việc lựa chọn gói và framework trước tiên thử tệp nhị phân `Electron Framework` cấp cao nhất, sau đó là `Versions/Current/Electron Framework`, rồi `Versions/A/Electron Framework`. Các liên kết tượng trưng chỉ được lần theo khi chúng vẫn nằm trong framework đó.
Trên Windows và Linux, truyền tệp thực thi Electron chính. Luồng cầu chì nằm trong tệp thực thi đó thay vì trong `app.asar`.

Kết quả Bằng chứng (Evidence) nội tuyến chứa:

- đường dẫn đã chọn, tệp thông thường đã phân giải, SHA-256, kích thước và định dạng vùng chứa của nó (`mach-o`, `mach-o-universal`, `pe`, hoặc `elf`);
- một mục slice cho tệp Mach-O, PE hoặc ELF mỏng, và một mục cho mỗi slice của tệp Mach-O phổ quát, với các giá trị `sentinel_offset` tuyệt đối của tệp;
- `wire_version`, `declared_length`, và các cầu chì đã sắp xếp `{ index, name, state, raw_byte }`;
- `sentinel_count`, bao gồm bằng 0 và nhiều hơn một;
- `runtime_implications`, mỗi cái được đánh dấu `relationship: "derivation"` và gắn với chỉ số cầu chì mà nó xuất phát.

Tên phiên bản 1, theo thứ tự chỉ số, là `RunAsNode`, `EnableCookieEncryption`, `EnableNodeOptionsEnvironmentVariable`, `EnableNodeCliInspectArguments`, `EnableEmbeddedAsarIntegrityValidation`, `OnlyLoadAppFromAsar`, `LoadBrowserProcessSpecificV8Snapshot`, `GrantFileProtocolExtraPrivileges`, và `WasmTrapHandlers`. Một chỉ số vượt quá bảng đó, hoặc bất kỳ chỉ số nào trên phiên bản wire khác, giữ `name: null` và byte thô.

Các byte wire là `0x30` vô hiệu hóa (disabled), `0x31` bật (enabled), và `0x72` đã xóa (removed). Bất kỳ byte nào khác là `unknown`. Một cầu chì đã xóa không được bật: các cờ Inspector và `ELECTRON_RUN_AS_NODE` không được tôn trọng, và mã ứng dụng không bị giới hạn ở `app.asar`. Trên macOS và Windows, byte `EnableEmbeddedAsarIntegrityValidation` bị xóa hoặc vô hiệu hóa có nghĩa là tính toàn vẹn ASAR nhúng không được thực thi, và byte được bật có nghĩa là nó được thực thi. Electron chỉ biên dịch xác thực đó trên macOS và Windows. Trên tệp thực thi Linux ELF, byte quan sát được được bảo toàn và kết quả thực thi là `unknown`.

## Điểm chưa xác định và nhiều lính canh (Sentinels)

Không có lính canh (sentinel) có nghĩa là trạng thái cầu chì chưa xác định. Nó không có nghĩa là các cầu chì bị vô hiệu hóa. Một wire tiếp tục vượt qua slice được báo cáo là bị cắt cụt, chỉ với các byte cầu chì có mặt. Hai hoặc nhiều lính canh trong một slice đều được báo cáo. REA không tự ý chọn một cái.

Byte cầu chì không chứng minh rằng ứng dụng bật Inspector theo một cách nào khác, ví dụ thông qua `app.commandLine` hoặc chuyển đổi gỡ lỗi từ xa trong mã ứng dụng.

Tệp được chọn phải là tệp thông thường, hoặc gói phân giải thành tệp thông thường. Các liên kết tượng trưng chỉ được lần theo bên trong `Electron Framework.framework`. Bộ đọc tiếp nhận tối đa 1 GiB, tìm kiếm với các cửa sổ 256 KiB và giữ tối đa 32 lính canh trên mỗi slice. Cuộc kiểm tra này không tạo tệp tạm thời và không sửa đổi các byte đã chọn.
