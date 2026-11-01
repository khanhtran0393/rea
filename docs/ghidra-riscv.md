# Phân tích RISC-V ELF qua Ghidra

Luồng này mở rộng phân tích ELF native cho các mục tiêu `EM_RISCV`. Nó không thực thi mã RISC-V, giả lập chuỗi khởi động hoặc lưu trữ lại firmware. Nó giải quyết [#2063](https://github.com/morluto/rea/issues/2063) và tuân theo cùng dạng luồng như [hỗ trợ MIPS thông thường](ghidra-mips.md).

## Ranh giới tiếp nhận

ELF `EM_RISCV` (243) được giữ lại dưới dạng họ `riscv` trung lập với provider. Mục tiêu cũng giữ lại các khai báo ELF thô quyết định một luồng: class, thứ tự byte, `e_type`, `e_flags`, và chuỗi ISA mà `.riscv.attributes` khai báo.

Luồng Ghidra tiếp nhận các hình ảnh little-endian `ET_EXEC` không có `EF_RISCV_RVE`, `EF_RISCV_TSO`, hoặc bất kỳ bit nào ngoài `EF_RISCV_RVC | EF_RISCV_FLOAT_ABI`, trong một trong ba luồng:
- Các hình ảnh ELF64 hoặc ELF32 khai báo `EF_RISCV_FLOAT_ABI_DOUBLE` sử dụng mô hình double-float gốc của Ghidra không thay đổi (`RISCV:LE:64:default` hoặc `RISCV:LE:32:default` + `gcc`).
- Các hình ảnh ELF32 khai báo ABI soft-float (các bit float-ABI trong `e_flags` là `0`, ilp32) sử dụng chính sách quy ước gọi ilp32 của REA. Chúng cũng phải khai báo một cơ sở RV32I và chỉ các phần mở rộng đã giải mã trong `Tag_RISCV_arch`.

Các hình ảnh big-endian bị từ chối vì Ghidra 12.1.4 không cung cấp ngôn ngữ RISC-V big-endian.

## Lý do ghim ngôn ngữ

Lựa chọn dựa trên tiêu đề của Ghidra không đáng tin cậy cho RV32. Do đó REA ghim `RISCV:LE:32:default` hoặc `RISCV:LE:64:default` với thông số trình biên dịch `gcc` theo từng class.
