# Phân tích MIPS ELF qua Ghidra

Luồng này mở rộng phân tích ELF native; nó không thực thi mã MIPS hoặc lưu trữ lại firmware. Nó thúc đẩy phần MIPS thông thường của [#718](https://github.com/morluto/rea/issues/718). PSP/Allegrex có [profile static-ELF riêng biệt](ghidra-psp.md).

## Ranh giới tiếp nhận

ELF `EM_MIPS` được giữ lại dưới dạng họ `mips` trung lập với provider. Mục tiêu cũng giữ lại class, thứ tự byte, `e_type`, `e_flags` thô và các khai báo ABI đã kiểm tra.

Luồng Ghidra tiếp nhận ELF32 `ET_EXEC` và `ET_REL`, little hoặc big endian, với các cờ o32 và `EF_MIPS_ARCH_32R2` rõ ràng và không có cờ mã hóa đặc thù máy hoặc ASE. ELF64, n32/n64, ABI không xác định, microMIPS và các biến thể đặc thù máy không được bao phủ bởi luồng lệnh tiêu chuẩn này.

Phần tiếp nhận thứ hai bao gồm các hình ảnh firmware cũ không có bản ghi ([#1620](https://github.com/morluto/rea/issues/1620)): tiêu đề ELF32 `ET_EXEC` khai báo `EF_MIPS_ARCH_1` (MIPS-I, trường ISA bằng 0), `EF_MIPS_ARCH_32` hoặc `EF_MIPS_ARCH_32R2` với trường ABI chưa xác định. Luồng ghim `-processor MIPS:<LE|BE>:32:default`.
