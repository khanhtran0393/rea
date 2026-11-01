# TLS Callbacks và cấu hình nạp PE (PE TLS Callbacks & Load Configuration)

Sử dụng `inspect_pe_load_config` để xác định vị trí các TLS callback được khai báo và kiểm tra cấu hình nạp (load configuration) của một tệp ảnh PE32 hoặc PE32+ cục bộ rõ ràng. Không cần bộ tháo mã, hệ điều hành Windows, hay việc thực thi mục tiêu.

```sh
rea inspect-pe-load-config /duong/dan/tuyet/doi/toi/application.exe --json
rea inspect-pe-load-config /duong/dan/tuyet/doi/toi/application.exe --max-callbacks 256 --json
```

MCP chấp nhận `{ "path": "/duong/dan/tuyet/doi/toi/application.exe" }`. Đường dẫn CLI có thể tương đối so với thư mục hiện tại; đường dẫn MCP bắt buộc phải là tuyệt đối.

Bằng chứng (Evidence) hoàn chỉnh bao gồm đường dẫn hiện vật, SHA-256, phạm vi tệp thư mục và phạm vi tệp mảng callback. `tls` báo cáo phạm vi dữ liệu thô, địa chỉ chỉ mục, kích thước zero-fill, đặc tính và mảng callback kết thúc bằng null. VA của callback là chuỗi thập lục phân để bảo toàn độ chính xác 64-bit. RVA, độ lệch tệp đã ánh xạ và section là các vị trí được suy dẫn; các ánh xạ không xác định là `null`.

`load_config` chỉ báo cáo các trường khi `Size` đã khai báo của nó bao phủ toàn bộ trường. Các tệp ảnh MSVC x86 thường khai báo một thư mục dữ liệu load-config 64 byte trong khi `Size` của chính cấu trúc lớn hơn; giống như bộ nạp Windows, REA đọc các byte `Size` khi chúng được hỗ trợ bởi tệp trong một lần chạy, và báo cáo cả `directory.size` và `size`. `structure_location` là phạm vi tệp của các byte `Size` mà REA đã kiểm tra; các vị trí Bằng chứng bao gồm nó bên cạnh phạm vi thư mục đã khai báo khi cả hai khác nhau. Các trường không khả dụng là `null`; một con trỏ số 0 được bao phủ vẫn là `"0x0"`.

Báo cáo bao gồm security cookie, bảng/số lượng PE32 SafeSEH, con trỏ và bảng/số lượng CFG, bảng/số lượng IAT và long-jump đã lấy địa chỉ, bảng/số lượng EH continuation, và con trỏ kiểm tra XFG. Con trỏ tái định vị động và CHPE là các giá trị thô. Độ lệch bảng tái định vị động (DWORD) và section (WORD) được báo cáo riêng biệt khi `Size` bao phủ chúng, bao gồm cả các giá trị bằng 0; REA không suy diễn một VA từ cặp đó hoặc giải mã các mục bảng đích và bảng giảm thiểu rủi ro.

Các cờ Guard giữ lại DWORD thô, tên được nhận diện, các bit chưa biết và bước nhảy mục CFG. Các tuyên bố giảm thiểu rủi ro bắt nguồn từ những cờ đó mang `evidence_kind: "inferred"`; chúng không chứng minh việc thực thi trong thời gian chạy. Tệp ảnh không bao giờ được thực thi. Các thư mục bị thiếu là sự vắng mặt quan sát được của những khai báo đó, không phải bằng chứng cho thấy không có callback hoặc biện pháp giảm thiểu nào tồn tại bởi một cơ chế khác.

Ngân sách đọc mặc định là 64 MiB (`max_file_bytes`, mức trần 512 MiB). Ngân sách callback mặc định là 64 (`max_callbacks`, mức trần 4096). Một mảng ngân sách chính xác được chấp nhận nếu vị trí tiếp theo của nó là null. Bộ lưu trữ thư mục và mảng callback bị cắt cụt, mơ hồ hoặc chưa được ánh xạ sẽ thất bại đóng (fails closed), tương tự như khi cạn kiệt ngân sách; không có thành công một phần nào được trả về. Một **đích** callback chưa được ánh xạ vẫn giữ lại VA/RVA của nó với độ lệch tệp chưa xác định. Các phiên bản load-config vượt quá các trường được hỗ trợ vẫn chưa được giải mã.

Tài liệu tham khảo bố cục: [Định dạng Microsoft PE](https://learn.microsoft.com/en-us/windows/win32/debug/pe-format), [IMAGE_LOAD_CONFIG_DIRECTORY32](https://learn.microsoft.com/en-us/windows/win32/api/winnt/ns-winnt-image_load_config_directory32), và [IMAGE_LOAD_CONFIG_DIRECTORY64](https://learn.microsoft.com/en-us/windows/win32/api/winnt/ns-winnt-image_load_config_directory64).

Đối với một fixture biên dịch chéo thực tế, chạy `npm run verify:pe:load-config` với Zig trên PATH (hoặc `REA_ZIG` được đặt thành đường dẫn của nó). Script này xây dựng các mẫu PE32 và PE32+ và kiểm tra chúng thông qua CLI công khai mà không cần chạy chúng.
