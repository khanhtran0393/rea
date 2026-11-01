# Windows Ghidra P0

Hướng dẫn này mô tả nhánh main của kho lưu trữ. Hỗ trợ PE x86 (32-bit) native có sẵn trên main; các bản phát hành npm lên đến 5.0.0 tiếp nhận các mục tiêu PE x86-64 trên Windows.

Trạng thái: hỗ trợ thử nghiệm Windows x64 cho ranh giới P0 chỉ đọc. Các bản dựng gói Windows đi kèm các điều khiển tiến trình native, hệ thống tệp và DACL. REA sử dụng chúng tự động khi nhà phân tích chọn một thao tác Ghidra; không yêu cầu cờ cấp quyền bổ sung hoặc chế độ suy giảm.

## Ranh giới được hỗ trợ

- Windows 10+ x64 với các phân vùng NTFS cục bộ cố định cho mục tiêu và tạm thời;
- Node.js 22.x (>=22.23.4), 24.x (>=24.22.1), hoặc 26.12.1+;
- Bản phân phối Ghidra 12.x (>=12.1) do người vận hành cài đặt (đã xác minh với 12.1.4);
- JDK 64-bit đầy đủ nằm trong dải Java đã khai báo của bản cài đặt đó (JDK 21 trở lên cho các bản phát hành 12.1 hiện tại);
- Ứng dụng/DLL PE native, không phải managed x86, x86-64, ARMNT (Thumb-2) hoặc ARM64, hoặc [hình ảnh raw NMOS 6502](raw-binary.md#nmos-6502) với điểm nhập do bên gọi cung cấp; và
- 27 thao tác kiểm kê, bộ nhớ và phân tích hàm Ghidra chỉ đọc.

Các lần đọc bộ nhớ đã nạp và độ lệch tệp bảo toàn các ánh xạ nguồn quan sát được của Ghidra. Kiểm tra hình ảnh nạp PE trả về trạng thái chứng thực không hỗ trợ rõ ràng.

Đối với NativeAOT RTR 9.1 PE x64, trình phân tích cú pháp snapshot chỉ đọc có thể báo cáo metadata thông qua `inspect_native_load_image` và `inspect_native_data_type` mà không cần JAR hay thay đổi cơ sở dữ liệu: các mối quan hệ MethodTable, slot ảo và chuỗi đóng băng. `analyze_function` dịch ngược các mục tiêu slot. Việc đọc cơ sở dữ liệu đối với các byte được bù nước (`read_bytes`, `search_strings`) không trả về gì trên máy chủ này. Xem [NativeAOT](ghidra-nativeaot.md).

REA không cài đặt hay nâng cấp Ghidra, Java, Python, npm, Node.js, Hopper hay trình biên dịch. Adapter sử dụng `HeadlessScript` Java đi kèm; Python và PyGhidra không phải là điều kiện tiên quyết.

## Cấu hình

```powershell
npm install --global rea-agents
$env:GHIDRA_INSTALL_DIR = "C:	ools\ghidra_12.1.4_PUBLIC"
$env:JAVA_HOME = "C:	ools\jdk-21"
$env:REA_ANALYSIS_PROVIDER = "ghidra"

rea doctor --json
rea providers --json
rea inspect "C:ixtures\sample.exe" --provider ghidra --format json
```

`rea doctor` phân biệt các điều kiện tiên quyết cài đặt với các lỗi artifact native hoặc hệ điều hành. Nó kiểm tra bản phát hành Ghidra, trình khởi chạy headless, JDK đầy đủ, máy chủ x64 và các điều khiển native.
