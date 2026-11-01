# Điều tra Tệp Nhị phân Native (Native Binary Investigation)

Tài liệu này hướng dẫn cách sử dụng REA để phân tích sâu các tệp nhị phân mã máy (PE, ELF, Mach-O) chạy trên các kiến trúc phần cứng x86, x64, ARM, ARM64, MIPS, RISC-V, và PowerPC.

## Bắt đầu với một Hàm (Start with a function)

Quy trình điều tra hiệu quả nhất bắt đầu từ một hàm mục tiêu (ví dụ: hàm `main`, hàm xử lý xác thực, hoặc hàm gọi API nghi vấn):

```sh
rea inspect-native ./target.exe --function "main" --decompile --json
```

Lệnh này trả về mã giả C (decompiled code), danh sách các khối cơ bản (basic blocks), và các tham số đầu vào/đầu ra của hàm.

## Bộ chọn CLI Chính xác (Exact CLI selectors)

REA hỗ trợ định vị chính xác hàm hoặc địa chỉ cần phân tích:
- Theo tên biểu tượng: `--function "VerifyLicense"`
- Theo địa chỉ bộ nhớ ảo: `--address "0x140001230"`
- Theo địa chỉ tương đối RVA: `--rva "0x1230"`

## Các Nguyên thủy Lệnh, Lời gọi và Kiểu Native

REA phân tích chi tiết:
- **Tập lệnh**: Phân tích giải mã từng lệnh assembly (x86/x64, ARM64) với đầy đủ toán hạng.
- **Lời gọi hàm**: Danh sách các hàm con được gọi từ hàm này (`callees`) và danh sách các hàm gọi tới nó (`callers`).
- **Kiểu dữ liệu**: Nhận diện cấu trúc con trỏ, struct, mảng và các biến cục bộ trên ngăn xếp (stack frame).

### Hành vi Phân tích của Hopper (Hopper analysis behavior)

Khi cấu hình `--provider hopper` trên macOS:
- Tận dụng cơ chế nhận diện kiểu dữ liệu và cấu trúc Objective-C / Swift nâng cao của Hopper.
- Trích xuất thông tin selector, class name và metadata runtime của Apple.

## Vết Tĩnh có Giới hạn (Bounded Static Traces)

Thực hiện phân tích luồng điều khiển tĩnh trong một phạm vi độ sâu nhất định:

```sh
rea trace-calls ./target.exe --from "main" --max-depth 3 --json
```

Giúp tác tử nắm bắt nhanh cây gọi hàm mà không bị lạc vào các thư viện tiêu chuẩn hoặc vòng lặp đệ quy.

## Kiểm tra Tĩnh (Static Inspection)

Trích xuất các thông tin cơ sở của tệp thực thi:
- **Tiêu đề tệp (File Headers)**: Machine type, EntryPoint, ImageBase, Subsystem.
- **Các phân vùng (Sections)**: `.text`, `.data`, `.rdata`, `.pdata`, `.rsrc` kèm theo cờ quyền hạn (Read/Write/Execute) và độ hỗn loạn entropy.
- **Bảng Import/Export**: Danh sách các thư viện DLL/so được liên kết động và các hàm API hệ điều hành được sử dụng.

## Quan sát Desktop Native (Native desktop observation)

Đối với các ứng dụng GUI trên Windows hoặc macOS:
- Quan sát cấu trúc cửa sổ, handle, danh sách điều khiển giao diện người dùng.
- Theo dõi các thông điệp sự kiện vòng lặp chính (`WndProc` trên Windows).

## Quan sát Lời gọi Native (Native call observation)

Kết hợp với các công cụ thu thập động như Frida để ghi lại các lời gọi hàm thực tế trong thời gian chạy:
- Các tham số truyền vào hàm và giá trị trả về.
- Dấu vết ngăn xếp thời gian thực (Call stack trace).

## Ranh giới Nhà cung cấp và Xác minh

REA cho phép chuyển đổi linh hoạt giữa các backend:
- `ghidra`: Độ chính xác cao nhất trong dịch ngược mã giả C phức tạp.
- `ida`: Tích hợp với môi trường chuyên nghiệp có sẵn cơ sở dữ liệu `.idb`.
- `internal`: Phân tích tĩnh tức thì không cần cài đặt phần mềm bên ngoài.

## Độ bao phủ Phân cấp Interface Builder

Đối với các ứng dụng macOS và iOS:
- Phân tích cú pháp tệp `.nib` và `.storyboard`.
- Khôi phục cấu trúc phân cấp đối tượng giao diện người dùng, ràng buộc dữ liệu và liên kết hành động (outlets/actions).

## Độ chính xác Số nguyên trong Keyed Archive

Giải mã chính xác các tệp lưu trữ nhị phân của Apple (`NSKeyedArchiver`), bảo toàn độ rộng bit của các số nguyên 64-bit và số thực dấu phẩy động.

## Định vị Chữ ký Byte (Locate a byte signature)

Tìm kiếm các mẫu byte (pattern scanning / AOB scan) có chứa ký tự đại diện (wildcards):

```sh
rea scan-bytes ./binary --pattern "48 89 5C 24 ?? 57 48 83 EC 20" --json
```
