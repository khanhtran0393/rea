# Phân tích DOS MZ và COM qua Ghidra

REA có thể phân tích các tệp thực thi DOS MZ và các hình ảnh COM được chọn rõ ràng thông qua adapter Ghidra tự cung cấp. Các luồng xác minh Linux x64 và macOS arm64 thực hiện việc dịch mã (disassembly) 16-bit và dịch ngược (decompilation) thực tế thông qua CLI và stdio MCP. Tiếp nhận provider chấp nhận Ghidra 12.x (>=12.1) và dải JDK đã khai báo của bản cài đặt đó (Ghidra 12.1.4 và JDK 21); không yêu cầu trình giả lập DOS hay trình biên dịch chéo.

## Mở và kiểm tra

```bash
rea inspect /absolute/path/to/legacy.exe --provider ghidra --json
rea search /absolute/path/to/legacy.exe entry --kind procedures --provider ghidra --json
rea instructions /absolute/path/to/legacy.exe entry --provider ghidra --json
rea function /absolute/path/to/legacy.exe entry --provider ghidra --json
rea inspect-native-load-image /absolute/path/to/legacy.exe --provider ghidra --json
rea read-bytes /absolute/path/to/legacy.exe 0x10000 --length 16 --provider ghidra --json
rea address-to-file-offset /absolute/path/to/legacy.exe 0x10000 --provider ghidra --json
```

MCP sử dụng `open_binary`, `list_procedures`, `procedure_info`, `read_function_instructions`, và `analyze_function` với cùng ngữ nghĩa mục tiêu và provider đã tiếp nhận.

Trình phân tích DOS MZ xác thực số lượng trang, độ dài module đã khởi tạo, ranh giới tiêu đề và bảng tái định vị, đích tái định vị, và điểm nhập đã khởi tạo bằng các byte cấu trúc có sẵn và độ dài tệp thực tế. Nó báo cáo các dữ kiện DOS đó tách biệt khỏi bằng chứng tiêu đề Windows.

Ghidra nạp snapshot mục tiêu riêng tư với `MzLoader`, ngôn ngữ `x86:LE:16:Real Mode`, và thông số trình biên dịch `default`. Các thiết lập này, phân đoạn nạp Ghidra cố định `0x1000`, và quy ước địa chỉ tuyến tính được cam kết trong profile phân tích. Việc đóng phiên sẽ xóa dự án và snapshot tạm thời.

## Hình ảnh COM không có tiêu đề (Headerless COM)

COM không có tiêu đề nhận dạng. Hãy chọn cách diễn giải của nó một cách rõ ràng:

```bash
rea inspect /absolute/path/to/legacy.com --target-format dos-com --provider ghidra --json
rea function /absolute/path/to/legacy.com 0x10100 --target-format dos-com --provider ghidra --json
rea inspect-native-load-image /absolute/path/to/legacy.com --target-format dos-com --provider ghidra --json
```

```json
{
  "name": "open_binary",
  "arguments": {
    "path": "/absolute/path/to/legacy.com",
    "format": "dos-com",
    "provider_id": "ghidra"
  }
}
```

Các tệp phải chứa từ 1 đến 65.280 byte: phân đoạn COM 64 KiB trừ đi tiền tố PSP 256 byte. Ghidra sử dụng `BinaryLoader` với ngôn ngữ real-mode và ánh xạ độ lệch tệp 0 tại `1000:0100` (địa chỉ tuyến tính `0x10100`). Một script tiền phân tích đi kèm thiết lập hàm entry và ngữ cảnh CS/DS/ES/SS `0x1000`.

## Địa chỉ và phạm vi hàm

Địa chỉ không gian mặc định được trả về là tọa độ byte tuyến tính. Địa chỉ phân đoạn:độ lệch (segment:offset) của Ghidra có giá trị tuyến tính `segment * 16 + offset`.
