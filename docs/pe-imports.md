# Bảng import và delay-load PE (PE Import and Delay-Load Tables)

`inspect_pe_imports` / `inspect-pe-imports` trả lời câu hỏi liệu một tệp ảnh PE32/PE32+ cục bộ đã chọn có khai báo bảng `IMAGE_DIRECTORY_ENTRY_IMPORT` hay không, DLL và hàm nào mà bảng đó đặt tên, bộ mô tả delay-load nào có mặt, và mã băm import tương thích với pefile (imphash) của bảng import tiêu chuẩn. Nó đọc tệp mà không cần thực thi và không yêu cầu Windows SDK hay bộ tháo mã.

```bash
rea inspect-pe-imports /duong/dan/tuyet/doi/toi/application.exe --json
```

CLI phân giải các đường dẫn tương đối dựa trên thư mục làm việc của bên gọi. MCP và lược đồ đầu vào được công bố yêu cầu đường dẫn hệ thống tệp tuyệt đối và bảo toàn lựa chọn chính xác đó:

```json
{
  "name": "inspect_pe_imports",
  "arguments": { "path": "/artifacts/application.exe" }
}
```

## Quan sát (Observations)

Kết quả Bằng chứng (Evidence) nội tuyến chứa đường dẫn hiện vật, SHA-256 và độ dài byte. `import_directory` và `delay_import_directory` báo cáo RVA và kích thước của thư mục dữ liệu, hoặc `null` khi thư mục bị thiếu hoặc bằng 0. `locations` của chúng là các khoảng tệp thực sự được đọc cho mảng bộ mô tả và phần kết thúc của nó. Khi các bộ mô tả liên tiếp ánh xạ tới các byte tệp không liên tục (ví dụ: qua ranh giới section), mỗi khoảng được báo cáo riêng biệt và mỗi khoảng trở thành phạm vi độ lệch tệp Bằng chứng riêng.

Nếu không thể kiểm tra bộ mô tả đầu tiên, khai báo thư mục vẫn hiện diện và không có `locations` nào được tiêu thụ, và độ bao phủ là một phần. Bộ mô tả, thunk và tên kết thúc bằng NUL vẫn nằm trong không gian RVA 32-bit của PE, bao gồm cả tệp ảnh PE32+; các hàng hợp lệ trước đó vẫn tồn tại qua một lần duyệt bộ mô tả bị tràn.

`machine` giữ lại giá trị COFF không dấu ban đầu. `architecture` giải mã các máy đã biết, bao gồm ghi đè hệ điều hành ReadyToRun khi một thư mục CLI được khai báo; khai báo đó không xác lập metadata CLR hợp lệ hoặc tính tương thích runtime. Các máy không được nhận diện giữ lại `pe-machine-<thập_phân>` trong khi bảng import của chúng vẫn có thể kiểm tra được. Một máy được nhận diện có độ rộng mâu thuẫn với header tùy chọn là sai định dạng.

Mỗi mục trong `imports[]` báo cáo tên DLL, RVA của bộ mô tả và các hàm có tên hoặc số thứ tự (ordinal) với RVA của ILT/IAT. `delay_imports[]` báo cáo riêng các bộ mô tả delay-load dựa trên RVA. `imphash` là `get_imphash()` của pefile chỉ trên bảng import tiêu chuẩn; các hàng delay-load không tham gia. Import theo số thứ tự sử dụng bảng tên số thứ tự của pefile và nếu không thì dùng phương án dự phòng `ord<thập_phân>` của nó.

`imphash.md5` là `null` kèm theo `reason` bất cứ khi nào giá trị này không phải là mã băm pefile của tệp ảnh này: bảng import tiêu chuẩn là một phần, hoặc pefile sẽ đọc một chuỗi hàng khác (tên mà nó coi là không hợp lệ hoặc bị cắt bớt, số thứ tự 0, bộ mô tả không có hàm, hoặc bảng mà giới hạn của nó và giới hạn ký hiệu import cắt bớt). Các vấn đề về delay-load không giữ lại mã băm.

Một thư mục import bị thiếu là sự vắng mặt quan sát được của bảng đó. Đó không phải là bằng chứng tệp ảnh không bao giờ import API qua một bảng khác hoặc overlay.

## Giới hạn và lỗi (Limits & Failures)

Tệp được chọn được giới hạn ở 64 MiB theo mặc định (tối đa 512 MiB). Tối đa 256 bộ mô tả import và delay-load được kiểm tra theo mặc định (tối đa 4096), và tối đa 8192 hàm trên mỗi bảng (tối đa 262144). Header PE bị lỗi và ngân sách DLL hoặc hàm cạn kiệt sẽ thất bại mà không có thành công một phần. Tên DLL và hàm đã giải mã được giới hạn ở mức 16 MiB văn bản được lưu giữ cố định mà không có cơ chế ghi đè runtime; `max_file_bytes`, `max_dlls` và `max_functions` không nâng mức này lên. Lỗi tài nguyên báo cáo mức cho phép bị cạn kiệt dưới dạng `limit` (`dlls`, `functions` hoặc `retained_text`) kèm theo phương án khôi phục phù hợp.

Tên DLL không đọc được, các mục hint/name, hoặc thunk được báo cáo là độ bao phủ một phần kèm theo `issues`. Một bộ mô tả có RVA bảng bắt buộc bằng 0 (một `FirstThunk` tiêu chuẩn, hoặc một địa chỉ import delay-load hoặc bảng tên) được báo cáo là `malformed_entry` không có hàm, không bao giờ là một kho kiểm kê trống hoàn chỉnh. Các bộ mô tả delay-load lưu trữ địa chỉ ảo thay vì RVA được ghi lại là `unsupported_form` và không được giải mã.

Quy trình làm việc này không giải quyết các import ràng buộc (bound imports), nội dung IAT đã nạp, hoặc tên chuyển tiếp export trong một DLL khác.
