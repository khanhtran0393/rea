# Phân tích tĩnh PSP ELF qua Ghidra

Chuyên môn hóa này được xây dựng dựa trên [hỗ trợ MIPS thông thường](ghidra-mips.md) cho [#1330](https://github.com/morluto/rea/issues/1330). Nó tái sử dụng các quy trình làm việc CLI/MCP, cầu nối Ghidra, Bằng chứng và snapshot hiện có của REA. Các lệnh Allegrex, quy ước nạp và gọi hàm của PSP vẫn thuộc quyền sở hữu của [ghidra-allegrex](https://github.com/kotcrab/ghidra-allegrex).

## Ranh giới ban đầu

Profile ban đầu bị giới hạn ở Linux x64, little-endian ELF32 `ET_EXEC`, `EM_MIPS` và khai báo PSPSDK `e_flags=0x10a23001`. Các bit đó khai báo MIPS II, Allegrex, EABI32 và NOREORDER; chúng không phải là bằng chứng về ngữ nghĩa của mọi lệnh. Profile cũng yêu cầu một bản ghi cờ MIPS ABI phiên bản 0 đã được kiểm tra khai báo ISA level 2/revision 0, 32-bit GPR và CPR1, không có CPR2, dấu phẩy động phần cứng độ chính xác đơn (`fpAbi=2`), không có phần mở rộng ISA hoặc ASE, `flags1=1` và `flags2=0`. Đây là các khai báo được báo cáo độc lập bởi GNU readelf 2.44 của PSPDEV v20261001 và bản dump section 24 byte của nó. Các khai báo bị thiếu, chưa được kiểm tra, mâu thuẫn và các khai báo khác của trình tạo đều bị từ chối. Profile PSP không phải là ngoại lệ đối với chính sách MIPS32r2/o32 thông thường; cổng kiểm soát riêng đó không thay đổi. Bản sửa đổi profile v2 ghi lại ranh giới đã sửa đổi này.

PRX, hình ảnh có thể tái định vị/dùng chung, container PBP/ISO/CSO, giả lập runtime và lớp phủ mã trực tiếp nằm ngoài profile đầu tiên này. Các máy chủ và tổ hợp cờ khác không được tiếp nhận âm thầm. Tên tệp không xác định mục tiêu PSP.

## Tự cung cấp tiện ích mở rộng tương thích (BYO Extension)

Cài đặt tiện ích mở rộng trong bản phân phối Ghidra đã chọn tại `$GHIDRA_INSTALL_DIR/Ghidra/Extensions/ghidra-allegrex`, với `Module.manifest`, `extension.properties`, các tệp ngôn ngữ và các JAR trình nạp của nó. Bản cài đặt trong thư mục người dùng (user-home) không được tìm kiếm vì REA sử dụng thư mục phân tích cô lập. REA không bao giờ tải xuống, cài đặt, xây dựng lại hoặc đóng gói sẵn tiện ích mở rộng.

Thử nghiệm khả năng tương thích đã công bố sử dụng Ghidra **12.1.3**, tiện ích mở rộng **v21.4**, PSPDEV **v20261001** và JDK 21. Kết quả provider công khai mới phải được báo cáo tách biệt khỏi thử nghiệm cầu nối lịch sử đó. Thuộc tính `version` của tiện ích mở rộng khai báo **phiên bản Ghidra** tương thích của nó, không phải thẻ phát hành; thuộc tính này phải khớp với bản cài đặt đã chọn. Các bản dựng cùng phiên bản khác không được coi là đã xác minh độc lập chỉ vì chúng vượt qua kiểm tra metadata này. Một tiện ích mở rộng bị lỗi hoặc không thể nạp vẫn sẽ làm thất bại việc nạp thực tế.

REA lấy dấu vân tay các tệp tiện ích mở rộng đã cài đặt trước khi giải quyết profile phân tích và kiểm tra lại danh tính đó ngay trước khi khởi chạy. Các liên kết mềm, tệp không thông thường, các thành phần bị thiếu và metadata không tương thích đều bị từ chối. Việc kiểm tra được giới hạn ở 1.024 mục, 16 cấp thư mục và 64 MiB, với việc băm tệp dạng luồng và thuộc tính 64 KiB. Việc xây dựng lại hoặc thay đổi nội dung sẽ tạo ra một profile khác; một snapshot cũ không được âm thầm tái sử dụng với profile mới. Đây là định danh nội dung, không phải chữ ký hay hộp cát cho mã đã cài đặt. Không sửa đổi bản cài đặt Ghidra khi một phiên đang chạy.

## Giao diện công khai thông thường

```bash
rea analyze /absolute/path/to/psp.elf --provider ghidra --json
rea function /absolute/path/to/psp.elf function_name --provider ghidra --json
```

MCP sử dụng `open_binary` với đường dẫn tuyệt đối và `provider_id: "ghidra"`, sau đó thực hiện các thao tác kiểm kê và hàm thông thường. Bộ nạp rõ ràng là `PspElfLoader`, ngôn ngữ `Allegrex:LE:32:default`, trình biên dịch `default`. Bắt tay phải xác nhận ngôn ngữ và trình biên dịch trước khi cung cấp các quan sát. Các điều kiện tiên quyết bị thiếu sẽ trả về `PSP_EXTENSION_UNAVAILABLE` kèm theo biện pháp cài đặt; không có chuyển đổi dự phòng provider hoặc cài đặt tự động nào diễn ra.

Bằng chứng giữ lại họ kiến trúc `mips` và profile dành riêng cho PSP của nó, bao gồm danh tính nội dung tiện ích mở rộng. Nó cũng mang các giới hạn về VFPU và chỉ phân tích tĩnh. Tiện ích mở rộng không mô hình hóa hoàn toàn các hiệu ứng tiền tố VFPU; mã được dịch ngược không rỗng không phải là bằng chứng về hành vi vector chính xác. Việc chỉnh sửa metadata cục bộ trong phiên hiện có không làm thay đổi các byte thực thi hoặc biến một snapshot thành một dự án Ghidra lâu dài.

## Xác minh

Sử dụng bộ xác minh MIPS thực tế hiện có với tùy chọn PSP của nó. Các công cụ bắt buộc do bên gọi cung cấp là `make`, `psp-config`, `psp-gcc`, `psp-readelf`, `psp-objdump`, `PSPDEV`, `GHIDRA_INSTALL_DIR` và một `JAVA_HOME` tương thích.

```bash
npm run build:cached
node scripts/verify-real-ghidra-mips.mjs --psp
# Trong bản cài đặt dùng một lần không có Allegrex, kiểm tra các từ chối công khai thực tế:
node scripts/verify-real-ghidra-mips.mjs --psp-missing-extension
# MIPS thông thường phải duy trì tính chung khi Allegrex được cài đặt:
npm run verify:ghidra:mips
```

Bộ xác minh thông thường cũng kiểm tra xem một bản sao firmware bị cắt ngắn phần có bị từ chối khi mục tiêu được mở hay không. Kiểm tra đó chỉ chạy ở chế độ chung; báo cáo PSP ghi lại rõ ràng phạm vi carved-PSP là chưa chạy.
