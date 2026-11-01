# Phân tích nhị phân thô không có header với Ghidra (Headerless Raw-Binary Analysis)

Các tệp ảnh không có header được đề cập ở đây. Tệp ảnh GameCube/Wii DOL có header và sử dụng lựa chọn định dạng riêng; xem [phân tích GameCube DOL](ghidra-dol.md).

REA tiếp nhận một tệp thực thi không có header khi bên gọi cung cấp họ CPU và địa chỉ nơi file offset 0 được ánh xạ. Việc diễn giải đó không được suy diễn từ phần mở rộng `.bin` hoặc từ các byte không được nhận diện.

## Mở và kiểm tra (Open & Inspect)

```bash
rea inspect /duong/dan/tuyet/doi/toi/kernel.img --target-format raw-binary --architecture arm --entry-point 0x8000 --provider ghidra --json
rea function /duong/dan/tuyet/doi/toi/kernel.img 0x8000 --target-format raw-binary --architecture arm --entry-point 0x8000 --provider ghidra --json
rea inspect-native-load-image /duong/dan/tuyet/doi/toi/kernel.img --target-format raw-binary --architecture arm --entry-point 0x8000 --provider ghidra --json
```

```json
{
  "name": "open_binary",
  "arguments": {
    "path": "/artifacts/kernel.img",
    "format": "raw-binary",
    "architecture": "arm",
    "entry_point": "0x8000",
    "provider_id": "ghidra"
  }
}
```

Ví dụ trên ánh xạ tệp ảnh trạng thái ARM 32-bit tại `0x8000`. Đối với `arm`, việc giải mã bắt đầu tại điểm vào ở trạng thái ARM (A32), do đó các tệp ảnh chỉ dùng Thumb, bao gồm firmware Cortex-M, không được hỗ trợ. Xem [Giới hạn](#limitations).

`architecture` là một trong các giá trị: `x86`, `x86_64`, `arm`, `arm64`, `6502`, hoặc `m68k`. `entry_point` là một địa chỉ thập lục phân có dạng `0x…`. Toàn bộ tệp được ánh xạ bắt đầu tại địa chỉ đó, và Ghidra sẽ tạo hàm `entry` ban đầu tại đó trước khi tự động phân tích.
Điểm vào ARM và AArch64 phải được căn chỉnh 4 byte; điểm vào m68k phải được căn chỉnh 2 byte; điểm vào x86 và 6502 có thể căn chỉnh theo byte.
Đối với 6502, điểm vào và toàn bộ tệp được ánh xạ phải nằm trong khoảng `0x0000..0xffff`; các tệp ảnh vượt qua ranh giới 16-bit sẽ bị từ chối trước khi khởi động provider.
Đối với m68k, ánh xạ phải nằm trong khoảng `0x0..0xffffffff`.
`open_binary` và một `binary_session` mở trả về `entry_point` chuẩn tắc cho các mục tiêu thô để bên gọi có thể xác minh trực tiếp ánh xạ đã chọn.
Bỏ qua `format` sẽ giữ nguyên phát hiện dựa trên header; cung cấp architecture và entry mà không có `format=raw-binary` sẽ bị từ chối.

Ghidra import một snapshot riêng tư với `BinaryLoader` và các ngôn ngữ chung cùng đặc tả trình biên dịch sau. Mỗi họ có một thứ tự byte cố định: m68k là big-endian và các họ khác là little-endian.

| architecture | language_id | compiler_spec_id |
| :--- | :--- | :--- |
| x86 | `x86:LE:32:default` | `gcc` |
| x86_64 | `x86:LE:64:default` | `gcc` |
| arm | `ARM:LE:32:v7` | `default` |
| arm64 | `AARCH64:LE:64:v8A` | `default` |
| 6502 | `6502:LE:16:default` | `default` |
| m68k | `68000:BE:32:default` | `default` |

Tài liệu tham khảo bố cục: [Phân tích Ghidra native](ghidra-provider.md).
