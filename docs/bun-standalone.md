# Tệp thực thi độc lập Bun (Bun Standalone Executables)

`inspect-artifact` kiểm kê đồ thị mô-đun Bun được nhúng trong Mach-O `__BUN,__bun`, PE `.bun`, ELF `.bun`, và các phần đuôi (trailers) được gắn thêm. `extract-artifact` ghi nội dung mô-đun được lưu trữ của chúng thông qua cây đầu ra an toàn hiện có. Cả hai thao tác đều không thực thi tệp thực thi hoặc dịch ngược bytecode.

```sh
rea inspect-artifact ./hello --json
rea extract-artifact ./hello --json
```

Trong MCP, mở tệp thực thi bằng `open_binary`, sau đó gọi `inspect_artifact`.
Sử dụng `detail: "summary"` cho các đồ thị lớn và `inspect_analysis_view` để phân trang Bằng chứng (Evidence) được lưu giữ. `entry_point_logical_path` của metadata gốc là vị trí thuộc quyền sở hữu của bộ đọc đối với mô-đun điểm vào (entry module) của nó. Sử dụng nó làm bộ lọc `logical_paths` trong kiểm kê, sau đó đọc các trường node đã chọn với danh tính nội dung được trả về. Điều này cũng hoạt động cho các tên được ánh xạ tới đường dẫn giải nén được đánh chỉ mục. Toàn bộ Bằng chứng vẫn được lưu giữ; bên gọi không cần chuẩn hóa tên nguồn.

Gốc báo cáo vật mang (carrier), bố cục, số lượng mô-đun, điểm vào, cờ thô và compile exec-argv. Mỗi mô-đun giữ lại tên nguồn, các giá trị loader và mã hóa, trạng thái điểm vào, kích thước và SHA-256. Sourcemaps, bytecode và các vùng module-info là các phần tử con riêng biệt. Tên nguồn như `/$bunfs/root/main.js` vẫn nằm trong metadata; việc giải nén sử dụng đường dẫn tương đối. Tên không an toàn hoặc không phải UTF-8 nhận một đường dẫn được đánh chỉ mục và giữ lại cách viết nguồn hoặc byte của chúng. Tên ảo sâu hơn 256 phần đường dẫn logic cũng sử dụng đường dẫn được đánh chỉ mục với một giới hạn rõ ràng, ngăn chặn việc xử lý cây đường dẫn không giới hạn trong khi vẫn giữ lại dữ kiện nguồn. Các tên trùng lặp vẫn là các lần xuất hiện riêng biệt; các quy tắc va chạm giải nén thông thường sẽ được áp dụng.

Các hình dạng bản ghi được nhận diện là bố cục 36 byte của Bun 1.3.3 và bố cục 52 byte của Bun 1.3.9. Các nhãn này xác định các bố cục cấu trúc, không phải phiên bản trình biên dịch được phát hiện. Bun không lưu trữ định danh bản sửa đổi rõ ràng: kích thước bảng không xác định và bố cục đã biết mơ hồ là không được hỗ trợ. Một bố cục tương lai tương thích theo từng byte không thể phân biệt được. Con trỏ không hợp lệ và các section được khai báo bị cắt cụt là sai định dạng. Việc thiếu phần đuôi mà không có section được đặt tên không xác lập đồ thị Bun. Các giá trị enum không xác định vẫn ở dạng thô, với tên là null.

Sourcemaps là các vùng Bun được lưu trữ, không nhất thiết là sourcemap JSON độc lập. Nội dung được giải nén giữ nguyên mã hóa ban đầu của chúng. Chỉ sử dụng phân tích JavaScript trên các tệp nguồn tương thích; bytecode được giữ lại làm bằng chứng, không được giải mã.

Phân tích cú pháp mô-đun đọc các phạm vi metadata chính xác thay vì tải các phần thân tài sản được lưu trữ hoặc toàn bộ các trang xung quanh các con trỏ nhỏ. Một bộ nhớ đệm giữ tối đa 256 KiB và 1.024 phạm vi cho một lần chuẩn bị của bộ đọc. Các lần đọc metadata không lưu vào bộ nhớ đệm tích lũy được giới hạn ở 256 MiB, được kiểm tra trước khi cấp phát hoặc I/O. Bảng mô-đun, chuỗi đã giải mã và cây đường dẫn logic kết hợp có các kiểm tra tiếp nhận riêng biệt; tối đa 250.000 tiền tố đường dẫn riêng biệt được tiếp nhận trước khi xây dựng kiểm kê. Các phần thân được lưu trữ được truyền theo phạm vi để băm và giải nén. Quan sát header/phần đuôi chỉ được tái sử dụng cho cùng một trạng thái tệp được tiếp nhận, với các kiểm tra đường dẫn đã chọn trước khi xuất bản.

Xem [giới hạn và khôi phục](design/limits-and-recovery.md) để biết các mức trần tiếp nhận. Các bài kiểm tra tổng hợp bao gồm các vật mang và khai báo sai định dạng. Chạy `node scripts/verify-bun-standalone.mjs` sau `npm run build:cached` để xác minh Bun cục bộ, và thêm `--docker` để biên dịch một fixture Linux với `oven/bun:1.4.0`. Bun và Docker chỉ là các trình tạo fixture và không được REA cài đặt.

Tài liệu tham khảo bố cục: [Bộ tuần tự hóa Bun 1.3.3](https://github.com/oven-sh/bun/blob/bun-v1.3.3/src/StandaloneModuleGraph.zig) và [Bộ tuần tự hóa Bun 1.3.9](https://github.com/oven-sh/bun/blob/bun-v1.3.9/src/StandaloneModuleGraph.zig).
