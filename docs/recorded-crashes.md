# Bằng chứng sự cố đã ghi lại (Recorded Crash Evidence)

`inspect_recorded_crash` / `rea inspect-recorded-crash` kiểm tra một tệp core dump ELF64 little-endian của Linux do bên gọi chọn độc lập với mục tiêu nhị phân đang hoạt động. Báo cáo sự cố Apple `.ips` sử dụng luồng [Apple IPS](apple-ips.md) riêng biệt. Hai profile kiến trúc được diễn giải là x86-64 và AArch64; mỗi profile liên kết các thanh ghi với bố cục `NT_PRSTATUS` của riêng nó và bất kỳ `e_machine` nào khác đều bị từ chối dưới dạng `unsupported_target`.

```sh
export REA_PWNTOOLS_PYTHON=/absolute/existing/venv/bin/python
rea inspect-recorded-crash ./crash.core --json
```

Python đã chọn cần pwntools 4.15.0, pyelftools 0.33 và Unicorn 2.1.2 chưa sửa đổi. REA không cài đặt các phụ thuộc này. Cấu trúc ELF gốc của ELFFile và ctypes Linux amd64 và aarch64 gốc của pwntools cung cấp khả năng phân tích cú pháp và bố cục ABI.

## Các quan sát được ghi lại

- Đường dẫn artifact, SHA-256 và kích thước gốc; kết quả chuẩn hóa đầy đủ và Bằng chứng đều ở dạng nội dòng qua CLI và MCP.
- Các tiêu đề chương trình gốc và các dải tệp/bộ nhớ phân đoạn được ghi lại. Bộ nhớ chưa được kết xuất và danh tính thực thi/thư viện hiện tại vẫn là unknown.
- Mọi danh tính ban đầu của note, các byte owner/descriptor và dải nguồn, bao gồm cả các owner và loại note lạ. Nhãn note tượng trưng CORE chỉ áp dụng cho owner `CORE\0` chính xác.
- Mọi note Linux CORE PRSTATUS chính xác dưới dạng một luồng riêng biệt được ghi lại, với PID lịch sử, tín hiệu và các thanh ghi uint64 không mất mát được liên kết với các byte nguồn (27 thanh ghi cho amd64, 34 cho AArch64).
- Số/mã/errno của SIGINFO có dấu. Ban đầu chỉ có các mã SIGSEGV 1/2 xác lập union địa chỉ lỗi; các biến thể khác giữ lại byte thô và ý nghĩa unknown.

## Ngữ cảnh ánh xạ trình gỡ lỗi tùy chọn (Optional Debugger Context)

```sh
export REA_PWNDBG_GDB=/absolute/existing/gdb
export REA_PWNDBG_GDBINIT=/absolute/unchanged/pwndbg/gdbinit.py
export REA_PWNDBG_VENV_PATH=/absolute/existing/pwndbg-venv
rea inspect-recorded-crash ./crash.core --debugger-context --json
```

MCP sử dụng `{ "path": "/artifacts/crash.core", "include_debugger_context": true }`. Việc bỏ qua facet này chỉ yêu cầu bằng chứng đã ghi lại. GDB mặc định là `/usr/bin/gdb`; plugin và môi trường ảo yêu cầu các đường dẫn tuyệt đối rõ ràng. Xác minh cục bộ ban đầu bao gồm Linux x64, GDB 13.1 và pwndbg 2026.09.15.

Tên ánh xạ là metadata hiển thị của trình tạo, với danh tính tệp hiện tại là unknown. Quyền ánh xạ được dẫn xuất từ các cờ được báo cáo. Một phong bì Bằng chứng chứa các ánh xạ này là `derived`; việc kiểm tra không có ngữ cảnh trình gỡ lỗi là `observed`.
