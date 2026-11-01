# Module hạt nhân Linux (Linux Kernel Modules)

`inspect_kernel_module` / `inspect-kernel-module` đọc metadata từ một đối tượng có thể tái định vị ELF32/ELF64 cục bộ được chọn (`.ko`) mà không cần thực thi nó. Metadata đó là các chuỗi khóa/giá trị `.modinfo`, CRC biểu tượng `__versions`, sự hiện diện của các section `.gnu.linkonce.this_module` và `__ksymtab*`, kích thước tệp quan sát được của `.init.text` và `.exit.text`, cùng một trailer chữ ký được thêm vào tùy chọn. Lệnh không yêu cầu máy chủ Linux hoặc chương trình `modinfo`.

```bash
rea inspect-kernel-module /absolute/path/to/module.ko --json
```

CLI giải quyết các đường dẫn tương đối so với thư mục làm việc của bên gọi. MCP yêu cầu một đường dẫn hệ thống tệp tuyệt đối:

```json
{
  "name": "inspect_kernel_module",
  "arguments": { "path": "/artifacts/demo.ko" }
}
```

## Các quan sát

Kết quả Bằng chứng nội dòng chứa đường dẫn artifact, SHA-256 và độ dài byte. `applicability` chỉ là `applicable` đối với `ET_REL`. Các loại ELF khác là `not_applicable`: dữ kiện tiêu đề được điền và `.modinfo`, `__versions`, cùng trailer chữ ký không được kiểm tra.

Các mục `.modinfo` giữ nguyên thứ tự tệp. Các khóa trùng lặp như `alias` và `parm` được bảo toàn. Mỗi mục giữ khóa, giá trị, dải tệp và SHA-256 của nó. Giá trị là các byte sau dấu `=` đầu tiên. Phần đệm NUL giữa các chuỗi bị bỏ qua. Khóa và giá trị sử dụng phép chiếu Latin1 bảo toàn byte: mỗi byte nguồn trở thành cùng một điểm mã Unicode được đánh số. Chúng không được giải mã dưới dạng UTF-8. `raw_sha256` băm các byte mục gốc; `raw_hex` hiển thị các byte đó khi chúng vừa với giới hạn nội dòng.

Các mục `__versions` là `struct modversion_info`: bản ghi 64 byte có CRC là một `unsigned long` và tên chiếm các byte còn lại. ELF32 lưu trữ CRC 32-bit và tên 60 byte. ELF64 lưu trữ unsigned long 64-bit và tên 56 byte. CRC là số thập lục phân viết thường có đệm số 0, 8 hoặc 16 ký tự. `versions.status: "absent"` có nghĩa là không tìm thấy section `__versions` nào khi tên section đã hoàn chỉnh.

`.gnu.linkonce.this_module` được báo cáo là present, absent, hoặc unknown. `layout` luôn là `unknown` vì `struct module` thay đổi theo phiên bản kernel và không được giải mã. Các section `__ksymtab*` được hỗ trợ bởi tệp được liệt kê theo thứ tự tiêu đề section theo tên và dải tệp. `.init.text` và `.exit.text` báo cáo kích thước được hỗ trợ bởi tệp quan sát được.

Trailer chữ ký là magic 27 byte `~Module signature appended~
` đứng trước bởi `module_signature` 12 byte (`algo`, `hash`, `id_type`, `signer_len`, `key_id_len`, 3 byte đệm, và `sig_len` big-endian). Khi các trường tham số đó bằng 0 và `id_type` là PKCS#7 (`2`), các byte `sig_len` đứng trước là dải PKCS#7 và SHA-256 của nó. Dải không được phân tích cú pháp và không có tuyên bố chứng chỉ, người ký hoặc độ bao phủ mã băm nào được đưa ra. Một trailer vắng mặt là sự vắng mặt quan sát được, không phải bằng chứng cho thấy module chưa được ký.

## Giới hạn và lỗi

Tệp được chọn bị giới hạn ở 64 MiB theo mặc định (tối đa 512 MiB). Tối đa 8.192 mục `.modinfo` và `__versions` cộng với các section `__ksymtab*` được liệt kê được kiểm tra theo mặc định (tối đa 65.536). Vượt quá ngân sách đó sẽ thất bại mà không thành công một phần. Khóa dài hơn 256 byte và giá trị dài hơn 4.096 byte giữ nguyên dạng lỗi trên từng mục; trường vừa vặn vẫn được sao chép. Tên section `__ksymtab*` dài hơn 256 byte giữ dải của nó với `name: null` và `ksymtab_name_exceeds_limit`.
