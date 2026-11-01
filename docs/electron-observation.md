# Quan sát trang tệp Electron (Electron File-Page Observation)

REA có thể đính kèm vào endpoint CDP của Electron/Chromium và kiểm tra các trang renderer `file://` hiện có mà không cần đánh giá JavaScript hoặc gọi API Electron. Yêu cầu cung cấp endpoint trực tiếp; REA chỉ chấp nhận các endpoint HTTP loopback cổng theo chuỗi ký tự. Việc chọn một endpoint sẽ hiển thị mọi mục tiêu Electron mà nó phục vụ, bao gồm đường dẫn tệp cục bộ và metadata trang.

Bề mặt runtime thụ động này tách biệt khỏi quy trình phân tích tĩnh không cần mục tiêu [`analyze_javascript_application`](javascript-artifact-reconstruction.md).

## Ranh giới mục tiêu

Yêu cầu cung cấp endpoint loopback và ID mục tiêu để kiểm tra. REA chấp nhận các URL `file://` cục bộ không có host giải quyết thành các tệp thông thường, bao gồm các trang mà ứng dụng đóng gói phục vụ từ bên trong `app.asar`. Đối với các trang đó, thành phần đường dẫn `.asar` đầu tiên là một tệp thông thường là kho lưu trữ, phần còn lại phải đặt tên cho một tệp thông thường trong tiêu đề của nó.

## Quy trình làm việc

`list_electron_targets` trả về mọi mục tiêu đủ điều kiện do endpoint được cung cấp phục vụ trong một mảng nội dòng.

Đối với lệnh gọi tiếp theo của MCP, hãy truyền cùng một endpoint loopback và ID mục tiêu đã chọn tới `inspect_electron_page`:

```bash
rea list-electron-targets http://127.0.0.1:9223 --json
rea inspect-electron-page http://127.0.0.1:9223 TARGET_ID   --observation-ms 100 --json
```

Nội dung script bị loại trừ theo mặc định. Đặt `include_script_sources` trong yêu cầu để trả về các nguồn script của mục tiêu đã chọn.
