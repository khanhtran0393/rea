# Siêu dữ liệu IL2CPP (IL2CPP Metadata)

REA cung cấp khả năng phân tích tĩnh nâng cao cho các ứng dụng Unity biên dịch qua IL2CPP thông qua việc phân tích tệp nhị phân mã máy (`GameAssembly.dll`, `libil2cpp.so`) kết hợp với tệp siêu dữ liệu `global-metadata.dat`.

## MCP

Thông qua giao thức MCP, sử dụng công cụ `inspect_il2cpp`:

```json
{
  "binary_path": "./libil2cpp.so",
  "metadata_path": "./global-metadata.dat"
}
```

Kết quả trả về danh sách các lớp, phương thức, trường dữ liệu, thuộc tính và địa chỉ hàm mã máy tương ứng.

## CLI

Sử dụng lệnh CLI `inspect-il2cpp`:

```sh
rea inspect-il2cpp --binary ./GameAssembly.dll --metadata ./global-metadata.dat --json
rea inspect-il2cpp --binary ./libil2cpp.so --metadata ./global-metadata.dat --dump-headers ./headers/
```

## Các quan sát (Observations)

Quá trình phân tích khôi phục:
- Toàn bộ danh sách TypeDefinition, MethodDefinition, FieldDefinition.
- Danh sách chuỗi ký tự chuỗi nội suy (`string literals`).
- Cấu trúc giao diện và các lớp kế thừa.
- Thông tin về các Generic Method và Generic Type Arguments.

## Địa chỉ hàm mã máy Native (Native method addresses)

REA ánh xạ từng phương thức managed sang địa chỉ bộ nhớ thực tế (Virtual Address / RVA) trong tệp nhị phân native:
- Cho phép đặt breakpoint gỡ lỗi trực tiếp trong x64dbg, IDA Pro, hoặc Ghidra.
- Hỗ trợ tạo tập lệnh hook Frida tự động dựa trên tên phương thức và chữ ký tham số.

### Các thực thể phương thức Generic (Generic method instances)

Các phương thức generic có thể chia sẻ một thân hàm thực thi native duy nhất đối với các kiểu tham chiếu (code sharing), hoặc có các bản dựng riêng biệt đối với các kiểu giá trị (value types). REA theo dõi và ghi nhận đầy đủ các địa chỉ chuyên biệt hóa này.

## Tên hàm trong Ghidra (Ghidra function names)

REA có thể tạo tập lệnh Ghidra Python/Java để tự động đổi tên các hàm vô danh trong dự án Ghidra:

```sh
rea inspect-il2cpp --binary ./GameAssembly.dll --metadata ./global-metadata.dat --export-ghidra-script ./rename_script.py
```

Khi chạy trong Ghidra, tập lệnh này sẽ đổi tên hàng chục nghìn hàm thành tên phương thức có ý nghĩa (ví dụ: `PlayerController$$Update`).

## Các trường hợp thất bại thường gặp (Failures)

- **Không khớp phiên bản metadata**: Tệp `global-metadata.dat` sử dụng phiên bản cấu trúc mới hơn hoặc cũ hơn phiên bản REA hỗ trợ.
- **Bị mã hóa hoặc xáo trộn (Protected/Obfuscated)**: Tiêu đề metadata bị xóa, bảng chuỗi bị XOR, hoặc mã hóa tùy chỉnh của các công cụ bảo vệ Unity thương mại.
- **Tệp nhị phân bị nén hoặc đóng gói**: Tệp `.so` bị nén bằng UPX hoặc packer tùy chỉnh cần được giải nén trước khi phân tích.

## Ngoài phạm vi của công cụ này

REA tập trung vào phân tích cấu trúc tĩnh và khôi phục khai báo loại. Nó không trực tiếp dịch ngược mã máy thành mã C# cấp cao hoàn chỉnh (khuyên dùng kết hợp với Il2CppDumper hoặc Cpp2IL nếu cần trích xuất mã giả C#).

## Xác minh Nhà sản xuất Thực tế (Real producer verification)

Bộ kiểm thử của REA được xác minh liên tục đối với các bản phát hành Unity chính thức từ phiên bản 2018.x đến 2023.x trên nhiều kiến trúc vi xử lý (x86, x64, ARM, ARM64).
