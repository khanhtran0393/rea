# Chẩn đoán nhị phân ngoại tuyến (Offline Binary Diagnostics)

`inspect_binary_layout` / `inspect-binary-layout` trả lời những gì một tệp rõ ràng chứa: bố cục section/segment, biểu tượng/tái định vị ban đầu, tên liên kết tĩnh và các chỉ báo giảm thiểu rủi ro (mitigation indicators). Lệnh hoạt động mà không cần mục tiêu Hopper, Ghidra hay IDA đang hoạt động.

## Engine tự cung cấp

Xác minh thực tế bao gồm các máy chủ **Linux x64** và các đối tượng **EXEC/DYN/REL** trong các profile: ELF64 x86-64, ELF32 i386, ELF32 little-endian ARM và ELF32 little- hoặc big-endian MIPS. Các máy, class và thứ tự byte khác, bao gồm AArch64 và ARM big-endian, trả về `unsupported_target`. Cung cấp một tệp thực thi Python tuyệt đối có môi trường chứa pwntools 4.15.0, pyelftools 0.33 và Unicorn 2.1.2:

```sh
REA_PWNTOOLS_PYTHON=/absolute/isolated-env/bin/python   rea inspect-binary-layout ./selected.elf --json
```

```json
{
  "name": "inspect_binary_layout",
  "arguments": { "path": "/artifacts/selected.elf" }
}
```

REA không bao giờ cài đặt Python, các gói hay GDB, không thay đổi tệp init của người dùng, không khởi chạy đối tượng đã chọn dưới dạng tiến trình máy chủ, hoặc yêu cầu giải quyết thư viện runtime.

## Diễn giải kết quả

- Đường dẫn artifact, SHA-256 và kích thước xác định các byte đã chọn ban đầu. Tất cả các phân tích đều đọc một snapshot ổn định thuộc sở hữu; bản gốc không thay đổi.
- Địa chỉ, độ lệch, độ dài và các cờ là các chuỗi thập lục phân.
- Mỗi biểu tượng giữ lại bảng và chỉ mục mục của nó.
- Các đối tượng có thể tái định vị có các tái định vị tương đối theo section và không có tuyên bố entry thực thi.
- Canary, PIE, NX, ngăn xếp thực thi (executable-stack) và RELRO là các suy diễn tĩnh.
