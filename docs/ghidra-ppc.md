# Phân tích PowerPC ELF qua Ghidra

Luồng này mở rộng phân tích ELF native sang PowerPC 32-bit big-endian; nó không thực thi mã PowerPC. Đây là cột mốc đầu tiên của [#1741](https://github.com/morluto/rea/issues/1741). Các tệp thực thi GameCube/Wii DOL được đề cập trong [Phân tích GameCube DOL](ghidra-dol.md). PPC64 và các biến thể PowerPC khác vẫn là các phần phát triển tiếp theo.

## Ranh giới tiếp nhận (Admission Boundary)

ELF `EM_PPC` (20) được giữ lại dưới dạng họ `powerpc` trung lập với provider. Mục tiêu cũng giữ lại class, thứ tự byte, `e_type` và `e_flags` thô. Việc nhận diện không có nghĩa là mọi provider hay biến thể PowerPC đều được hỗ trợ. `EM_PPC64` (21) không được nhận diện là họ này.

Luồng Ghidra chỉ tiếp nhận ELF32 `ET_EXEC`, big endian. ELF64, little-endian, các đối tượng có thể tái định vị (relocatable) và dùng chung (shared objects) bị từ chối với ràng buộc không hỗ trợ cụ thể. `EF_PPC_EMB` khai báo biến thể nhúng và bị từ chối; các bit `e_flags` còn lại chỉ khai báo quy ước tái định vị và không làm thay đổi việc giải mã lệnh.

## Ngôn ngữ liên kết (Bound Language)

Trình nạp ELF của Ghidra chọn `PowerPC:BE:32:e500:default` cho tiêu đề `EM_PPC` thông thường với `e_flags` bằng 0. e500 là biến thể Book-E nhúng có đơn vị SPE thay thế đơn vị dấu phẩy động cổ điển, do đó nó diễn giải sai mã của họ 750 thông thường. Không có gì trong tiêu đề phân biệt hai biến thể này.

Do đó, luồng này nạp với `PowerPC:BE:32:default` và thông số trình biên dịch `default`, cam kết lựa chọn đó trong phân tích cấu hình là `language_selection: "provider-pinned-v1"`, và kiểm tra ngôn ngữ mà Ghidra báo cáo sau khi nạp. Một ngôn ngữ khác được thiết lập qua `REA_GHIDRA_LANGUAGE_ID` (và tùy chọn `REA_GHIDRA_COMPILER_SPEC_ID`) sẽ thay thế mặc định của luồng và được tiếp nhận dựa trên danh mục ngôn ngữ đã cài đặt như thường lệ; ví dụ `PowerPC:BE:32:e500` cho mã e500 thực tế.

Các dữ kiện tiêu đề PowerPC được cam kết trong profile phân tích để các thay đổi về diễn giải sẽ làm vô hiệu hóa các snapshot gắn với profile. Danh tính luồng hỗ trợ được đánh phiên bản độc lập với danh tính tệp. Bằng chứng, kết quả vòng đời và snapshot đã lưu bảo toàn họ `powerpc`. Các adapter Hopper và IDA không mặc nhiên có được hỗ trợ PowerPC. Ghidra P0 trên Windows vẫn chỉ dành cho PE x86/x86-64.

## Sử dụng

Với Ghidra do người dùng cài đặt và JDK tương thích trên máy chủ Linux/macOS được hỗ trợ:

```bash
rea analyze /absolute/path/to/ppc.elf --provider ghidra --json
rea function /absolute/path/to/ppc.elf function_name --provider ghidra --json
```

MCP sử dụng các công cụ hiện có, bao gồm `open_binary` với đường dẫn tuyệt đối và `provider_id: "ghidra"`. Không có danh mục công cụ hay trình giả lập riêng cho PowerPC nào được thêm vào.

## Xác minh

Fixture độc lập thuộc quyền sở hữu mã nguồn là `tests/conformance/c/ppc.c`. Luồng đa mục tiêu tùy chọn yêu cầu Clang do người dùng cung cấp có target PowerPC, LLD, GNU `readelf`, Ghidra và JDK tương thích; các kiểm tra máy chủ native thông thường không tự động tải các công cụ này. `REA_PPC_CLANG` chọn Clang và `REA_PPC_READELF` chọn GNU readelf (ví dụ `/opt/homebrew/opt/binutils/bin/readelf` trên macOS). Đây là các điều kiện tiên quyết lúc kiểm thử, không phải runtime provider của REA.

```bash
GHIDRA_INSTALL_DIR=/absolute/path/to/ghidra npm run verify:ghidra:ppc
```

Luồng này biên dịch fixture cho lõi họ 750 (`-mcpu=750`) và không bao giờ thực thi nó. GNU readelf độc lập cung cấp các dữ kiện tiêu đề và địa chỉ biểu tượng, được so sánh với bộ giải quyết mục tiêu REA sản xuất trước khi khởi động provider. Phiên bản, đầu ra lệnh có giới hạn và đường dẫn công cụ đã chọn được lưu lại trong báo cáo xác minh. Trình phân tích cú pháp cố ý chỉ nhận diện đầu ra GNU của fixture này, từ chối các trường bị thiếu, không rõ ràng hoặc không mong đợi thay vì thay thế bằng giá trị mặc định.

Các thao tác MCP công khai hiện có kiểm tra ngôn ngữ và thông số trình biên dịch **quan sát được** của Ghidra cùng các mã băm tệp đã nạp; việc nạp e500 sẽ làm luồng thất bại. Tệp `ppc-probe.S` thuộc mã nguồn cung cấp mã hóa `addi` cố định và phân nhánh có điều kiện: quá trình xác minh kiểm tra byte, độ dài, toán hạng tức thời và đích trực tiếp, độc lập với tên biến của bộ dịch ngược. Byte toàn cục và chuỗi được kiểm tra tại các địa chỉ biểu tượng được báo cáo độc lập.

Khám phá và dịch ngược qua CLI/MCP, các lệnh gọi trực tiếp, danh tính mục tiêu/profile/Bằng chứng, kiểm tra vòng lặp snapshot và dọn dẹp tiến trình cuối cùng vẫn được bao phủ đầy đủ. Sự hiện diện của mã giả (pseudocode) là kiểm tra độ sống (liveness), không phải bằng chứng về tính tương đương ngữ nghĩa. Việc giải mã lệnh và sự đồng thuận của trình đọc không xác lập runtime hay ngữ nghĩa dấu phẩy động hoàn chỉnh. Báo cáo các luồng thực tế đã chạy và để lại các xác minh chưa thực thi ở trạng thái chờ (pending).
