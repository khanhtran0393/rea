# Container U-Boot legacy uImage

`inspect-artifact` và `extract-artifact` nhận diện chữ ký legacy uImage dạng big-endian `0x27051956` bất kể tên tệp. Chúng đọc các byte được lưu trữ cục bộ mà không khởi động, thực thi, giải nén hay sửa đổi hình ảnh (image). FIT là một định dạng khác; phân tích FIT nằm ngoài trình đọc này.

Cùng các dữ kiện tiêu đề/bảng miền và trình đọc artifact phục vụ cả CLI và MCP. Gốc là một kho lưu trữ với kiến trúc thực thi chưa xác định. CPU, OS, loại, kiểu nén, địa chỉ nạp và địa chỉ entry trong header là các khai báo, không phải bằng chứng về tải trọng tương thích hay thực thi được. Các mã định danh số thô vẫn được cung cấp bên cạnh tên. Tên loại bao gồm mọi loại hình ảnh trong `include/image.h` đã ghim, sử dụng tên từ `mkimage -T` của U-Boot; `IH_TYPE_LOADABLE` (22) không có tên và được gọi là `loadable`. Mã định danh không nhận diện được có tên là `unknown`.

CRC32 của header và CRC32 của tải trọng khai báo phải được xác minh trước khi kiểm kê thành công. `payload.bin` chứa toàn bộ tải trọng được lưu trữ. Hình ảnh đa tệp (multi-file images) cũng hiển thị `image-0.bin`, `image-1.bin`, v.v. Tên thành phần này được sinh từ thứ tự bảng. Các thành phần tuân theo bảng kích thước big-endian kết thúc bằng số 0, với 4 byte đệm sau mỗi thành phần không phải là cuối cùng. Hình ảnh script (`boot.scr`) mang cùng một bảng, do đó văn bản script của chúng là `image-0.bin`. Dải nguồn vật lý của thành phần và mã băm nội dung giữ lại nguồn gốc chính xác. Trích xuất ghi lại toàn bộ tải trọng và mọi thành phần, do đó đầu ra đa tệp có thể gần gấp đôi kích thước đầu vào. Tải trọng bị nén vẫn được giữ nguyên trạng thái nén.

Trường tên cố định 32 byte được giữ lại dưới dạng `name_bytes_hex`. `name` chứa các byte UTF-8 trước ký tự NUL đầu tiên, hoặc null khi các byte đó không thể giải mã chính xác. Ví dụ: trình tạo có thể cắt ngắn một ký tự đa byte tại ranh giới 32 byte; các ký tự thay thế không xác lập tên nguồn hợp lệ.

## Ranh giới tài nguyên và dữ kiện được lưu giữ

Trình đọc chấp nhận tối đa 4.096 thành phần, đây là chính sách tài nguyên chứ không phải giới hạn tối đa của định dạng. Nó đọc tối đa 16.388 byte cho bảng kích thước và từ chối số lượng vượt quá dưới dạng giới hạn dung lượng trước khi quét CRC tải trọng còn lại. Băm danh tính artifact dùng chung có chi phí riêng. Cùng tiền tố đó tham gia vào tính toán CRC32 tăng dần native; không có bảng CRC theo byte tùy chỉnh hay bộ đệm toàn bộ tải trọng nào được giữ lại. Các lần đọc CRC và luồng thành viên còn lại sử dụng các đoạn 64 KiB. Các dữ kiện đã phân tích được lưu cache cho một trình đọc. Trạng thái vật mang (carrier) được kiểm tra trong quá trình chuẩn bị thư mục, khi mở thành viên và sau khi hoàn thành luồng; các hàm lấy metadata trả về các dữ kiện đã lưu giữ. Thao tác đóng kết hợp các pipeline thành viên đang hoạt động và xác thực cuối cùng trước khi giải phóng bộ mô tả; dọn dẹp thất bại vẫn được giữ lại để thử lại.

Ưu tiên bản tóm tắt khi chỉ cần một vài dữ kiện:

```sh
rea inspect-artifact firmware.bin --detail summary   --evidence_output firmware.evidence.json --json
rea extract-artifact firmware.bin --json
```

Với MCP, `inspect_artifact` với `detail: "summary"` giữ lại Bằng chứng (Evidence) hoàn chỉnh. Sử dụng `inspect_analysis_view` và `parent_evidence_id` của nó cho các đường dẫn logic chính xác, các lô ID xuất hiện/nội dung, các trường và trang đã chọn. Độ bao phủ và các giới hạn vẫn được đính kèm. Các khung nhìn (views) được lưu giữ là các quan sát lịch sử gắn với Bằng chứng đã lưu. Bằng chứng hoàn chỉnh vẫn có sẵn; các khung nhìn này không thu hẹp kho kiểm kê đã kiểm tra.

## Phạm vi xác minh

Kernel initramfs GL-AR150 của OpenWrt 24.10.0 công khai được kiểm tra đối chiếu với [mã băm đã công bố](https://downloads.openwrt.org/releases/24.10.0/targets/ath79/generic/sha256sums):
`027c60a7061462d8145b11efa23ab02b334acb9a23a215f9095860b632ace4ff`.
Các công cụ độc lập `mkimage`/`dumpimage` 2023.01 cung cấp oracle tạo và trích xuất. Container được tạo sử dụng các tệp ELF máy chủ thật, metadata hệ thống và byte gzip được lưu trữ; các nhãn ARM64, VxWorks hoặc RISC-V được khai báo của chúng không chứng minh khả năng tương thích khi khởi động/runtime. Định nghĩa nhận diện và căn chỉnh của U-Boot được ghim theo commit `ece349ade2973e220f524ce59e59711cc919263f`.

Các bài kiểm tra CLI/full/compact MCP từ mã nguồn và bản biên dịch bao gồm các dải thành phần đã căn chỉnh, mã nhận dạng/văn bản không xác định, đầu vào lỗi/bị cắt cụt, lỗi CRC, dung lượng, thay đổi nguồn, hủy bỏ, các lần đọc đang chờ xử lý, xác thực cuối cùng và thử lại việc dọn dẹp. Kết quả từ trình tạo thực tế, phép đo chi phí toàn bộ tác vụ và các giới hạn được báo cáo trong PR. Không có hành vi giải nén hay runtime mục tiêu nào được xác minh.
