# Phân tích Mã Quản lý .NET (Managed-code Analysis)

REA cung cấp khả năng phân tích tĩnh toàn diện cho các ứng dụng và thư viện thuộc nền tảng .NET (.NET Framework, .NET Core, .NET 5/6/7/8/9, Mono, Unity).

## Kiểm tra một Cụm mã nguồn (Inspect an assembly)

Sử dụng công cụ `inspect_assembly` qua MCP hoặc lệnh CLI:

```sh
rea inspect-assembly ./App.exe --json
rea inspect-assembly ./Library.dll --detail full --json
```

### Giá trị mặc định của hằng số trường (Field constant defaults)

REA phân tích bảng metadata `#Blob` và `#US` để trích xuất:
- Các hằng số nguyên thủy (int, string, bool, float).
- Các chuỗi ký tự khởi tạo mặc định.
- Các giá trị khởi tạo trường tĩnh (static field initializers).

### Tổng quan Metadata và các Hàng được Lưu giữ

Trích xuất thông tin từ các bảng siêu dữ liệu ECMA-335:
- `TypeDef`, `MethodDef`, `Field`, `Param`, `Property`, `Event`.
- Cấu trúc cây kế thừa và các interface được hiện thực hóa.
- Danh sách các thuộc tính tùy chỉnh (Custom Attributes).

### Tóm tắt Thành viên và các Hàng Thành viên được Lưu giữ

Báo cáo chi tiết về phương thức:
- Tên phương thức, cờ hiển thị (`public`, `private`, `internal`).
- Chữ ký kiểu trả về và danh sách tham số.
- Các thuộc tính bổ sung: `static`, `virtual`, `abstract`, `final`, `pinvokeimpl`.

#### Tóm tắt CLI và các hàng thành viên đã lưu

```sh
rea inspect-assembly ./App.dll --filter-namespace "MyCompany.Core" --show-methods --json
```

## Phạm vi được Hỗ trợ (Shipped scope)

- **Các kiểu nạp tài nguyên**: Trích xuất tài nguyên nhúng (`.resources`, WPF BAML, hình ảnh).
- **Chuỗi ký tự người dùng (User Strings)**: Quét toàn bộ bảng User String để tìm kiếm URL, chuỗi định dạng, thông báo lỗi và khóa API tiềm năng.
- **Các kiểu khai báo mục tiêu cạnh**: Theo dõi các tham chiếu tới các cụm mã ngoài (AssemblyRef, TypeRef, MemberRef).

## Mục tiêu Phân tích (Analysis Objective)

Hỗ trợ các tác tử hiểu rõ kiến trúc, giao diện lập trình, cơ chế bảo vệ và logic nghiệp vụ cốt lõi của phần mềm .NET mà không cần chạy ứng dụng.

## So sánh Thành viên (Member Comparison)

So sánh hai phiên bản cụm mã .NET để phát hiện thay đổi API hoặc cập nhật bảo mật:

```sh
rea diff-assembly ./v1/Core.dll ./v2/Core.dll --json
```

## Khôi phục Siêu dữ liệu NativeAOT

Đối với các tệp nhị phân .NET được biên dịch trước sang mã máy hoàn toàn (NativeAOT):
- REA phân tích các cấu trúc dữ liệu thời gian chạy của NativeAOT để khôi phục lại định danh kiểu và phương thức đã bị loại bỏ bảng ECMA-335 tiêu chuẩn.

## Quy tắc Ranh giới Managed / Native

Phát hiện và phân tích các điểm giao cắt giữa mã quản lý và mã native:
- Khai báo P/Invoke (`[DllImport]`).
- Các hàm COM Interop và `UnmanagedCallersOnlyAttribute`.
- Cấu trúc dữ liệu marshaling (`MarshalAsAttribute`).

## Quan hệ Đồ thị Ứng dụng (Application graph relations)

Xây dựng đồ thị gọi hàm (Call Graph) tĩnh giữa các lớp và phương thức để xác định luồng thực thi dữ liệu chính.

## Cấu trúc Bản ghi Bằng chứng (Evidence record shape)

Mọi kết quả phân tích đều tuân theo lược đồ Bằng chứng chuẩn với các chỉ số đo lường độ tin cậy và định danh nguồn rõ ràng.

## Dấu vân tay CIL đã giải mã (Decoded-CIL Fingerprint)

Tạo hàm băm ổn định cho mã bytecode CIL (Common Intermediate Language) của từng phương thức sau khi chuẩn hóa các offset lệnh jump, cho phép nhận diện các hàm giống nhau ngay cả khi bị đổi tên.

## Ranh giới Công cụ và Đóng gói

Hỗ trợ các gói ứng dụng .NET hiện đại:
- Single-file bundles (.NET Core / 5+ SingleFileHost).
- Gói ứng dụng tự giải nén.
- Gói xuất bản ReadyToRun (R2R) chứa cả mã CIL và mã máy biên dịch sẵn.

## Xác minh Tuân thủ (Conformance Verification)

REA được kiểm tra tự động đối với các bộ kiểm thử chuẩn của Microsoft .NET Runtime và các thư viện mã nguồn mở phổ biến.
