# Báo cáo sự cố Apple IPS (Crash Reports)

`inspect_apple_ips` / `inspect-apple-ips` trả lời liệu một tệp Apple `.ips` cục bộ được chọn có phải là báo cáo sự cố (chuỗi `bug_type` là `"309"`) hay không, những dữ kiện về tiến trình, ngoại lệ và chấm dứt nào mà báo cáo đó ghi lại, những luồng và hàng `usedImages` nào mà nó liệt kê, và các chuỗi của nó khớp với những lớp sự cố đã được tài liệu hóa nào. Lệnh đọc tệp mà không thực thi mục tiêu và không yêu cầu Xcode, atos, trình dịch ngược hay pwntools.

```bash
rea inspect-apple-ips /absolute/path/to/report.ips --json
```

CLI giải quyết các đường dẫn tương đối so với thư mục làm việc của bên gọi. MCP yêu cầu một đường dẫn hệ thống tệp tuyệt đối và bảo toàn lựa chọn chính xác đó:

```json
{
  "name": "inspect_apple_ips",
  "arguments": { "path": "/artifacts/report.ips" }
}
```

## Các quan sát (Observations)

Kết quả Bằng chứng nội dòng chứa đường dẫn artifact, SHA-256 và độ dài byte. `metadata` là đối tượng JSON ở dòng đầu tiên. `bug_type` vẫn là một chuỗi. Thân sự cố chỉ được phân tích cú pháp khi chuỗi đó là `"309"`. Các loại khác, bao gồm cả stackshot `"288"`, không trả về kết quả sự cố.

`process`, `exception`, `termination`, `faulting_thread`, `threads`, và `images` tái tạo các trường thân tài liệu khi hiện diện, bao gồm `name` của mỗi hình ảnh và `symbolLocation` của mỗi khung (dưới dạng `symbol_location`, độ lệch byte tính từ đầu `recorded_symbol`). `termination` giữ tiến trình chấm dứt dưới dạng `by_proc` và `by_pid` cùng các `flags` của nó; `process.parent_pid` giữ `parentPid` bên cạnh `parent`; và `metadata` giữ `build_version` và `platform`. Số nguyên giữ các ký tự chữ số thập phân và dạng thập lục phân, bao gồm các thanh ghi arm64e trên 2^53. Vai trò của khung (frame) tuân theo chỉ mục khung được ghi lại: hàng số 0 của luồng bị lỗi đã giải quyết là `faulting_instruction`, hàng số 0 của các luồng khác là `current_instruction`, và các hàng sau là `return_address`. Mỗi địa chỉ runtime sử dụng cơ sở hình ảnh và độ lệch của chính hàng đó. Cờ `inline` được ghi lại không cung cấp độ lệch bị thiếu hoặc cho phép mượn địa chỉ của hàng khác. Khi `faultingThread` bị thiếu hoặc không giải quyết thành một đối tượng luồng được ghi lại, vai trò hàng 0 là `unknown`; việc tra cứu được ghi lại vẫn có sẵn.

Một tiến trình được dịch ghi lại các thanh ghi của Rosetta trong đối tượng lồng nhau `threadState.rosetta`; mỗi thanh ghi được báo cáo dưới dạng `rosetta.<name>` và được tính vào ngân sách thanh ghi. Một [nhật ký ReportCrash `.ips` công khai](https://github.com/Mortennn/Dozer/issues/189) hiển thị `rosetta.tmp2`, `tmp1`, và `tmp0` bên cạnh các thanh ghi x86 thông thường. Fixture kiểm thử trích dẫn đoạn trích thanh ghi đó bên trong thân báo cáo được tạo; nó không tạo ra một sự cố Rosetta mới trong quá trình kiểm tra.

Các chuỗi JSON quan sát được giữ lại `""`; `null` có nghĩa là trường bị thiếu hoặc JSON `null`. Điều này bao gồm đường dẫn hình ảnh, biểu tượng khung, văn bản tiến trình và dạng chuỗi `osVersion` cũ hơn. Các khóa JSON không được liệt kê và tên thanh ghi chung cũng giữ nguyên cách viết trống. Kiến trúc hình ảnh trống vẫn được ghi lại là `""` trong khi `uuid_unknown` vẫn là true, vì cách viết đó không xác lập một kiến trúc. Một UUID trống là sai định dạng, do đó cách viết chính xác của nó xuất hiện trong `coverage.type_mismatches` và UUID chuẩn của nó vẫn là unknown. `process.os_version` giữ từ điển của trình tạo (`train`, `build`, `release_type`, `is_embedded`) hoặc dạng chuỗi cũ hơn. UUID hình ảnh là 8-4-4-4-12 viết hoa; UUID toàn số 0 hoặc `arch` bị thiếu hoặc trống là `uuid_unknown`. Các chữ cái `source` hình ảnh giữ nguyên ý nghĩa được tài liệu hóa (`P` tiến trình, `S` bộ nhớ cache dùng chung, `C` thư viện cache dùng chung, `K` kernel, `U` cache kernel, `T` text exec kernel, `A` tuyệt đối). Các khóa mà tài liệu không liệt kê vẫn ở trong `unlisted_keys` hoặc `unlisted_body_keys`.

`image_index` của mỗi khung tham chiếu đến mục của nó trong `images`. Kiến trúc hình ảnh, UUID và `uuid_unknown` được giữ lại một lần trên hình ảnh đó. Hãy giải quyết tham chiếu khi sử dụng các dữ kiện đó; một chỉ mục vắng mặt hoặc nằm ngoài phạm vi sẽ khiến hình ảnh trở thành unknown. Điều này tránh việc lặp lại văn bản hình ảnh cho mọi khung ngăn xếp.

Một giá trị hiện diện có kiểu JSON khác với kiểu được tài liệu hóa không bao giờ được coi là vắng mặt: nó được liệt kê trong `coverage.type_mismatches` cùng với đường dẫn của nó, kiểu dự kiến và kiểu quan sát được, và `coverage.status` trở thành `partial`. Thân sự cố không có `threads` hoặc `usedImages` cũng là `partial`. Các chuỗi UUID hình ảnh bị lỗi tương tự làm cho độ bao phủ trở thành một phần; cách viết chính xác của chúng được giữ lại trong `recorded_value` của mismatch, trong khi UUID hình ảnh chuẩn vẫn là unknown. Cả 32 chữ số thập lục phân và UUID có dấu gạch nối chuẩn đều được chấp nhận. Các tham chiếu luồng/hình ảnh chưa được giải quyết cũng làm cho độ bao phủ trở thành một phần. Tra cứu số nguyên bị từ chối giữ nguyên chuỗi thập phân chính xác trong `recorded_integer`, bao gồm cả các giá trị âm và các giá trị trên uint32. Các giá trị tra cứu uint32 hợp lệ vẫn còn trong `faulting_thread` hoặc `image_index` ngay cả khi hàng được tham chiếu không khả dụng.

`classification` chỉ khớp với các mã SIGKILL được tài liệu hóa (so sánh theo dạng thập lục phân chuẩn của chúng), các kiểu phụ `EXC_GUARD`, các hậu tố xác thực con trỏ (pointer-authentication), và các chuỗi thư viện bị thiếu của dyld. Một báo cáo không khớp với bất kỳ quy tắc nào trong số đó là `unclassified`.

Các liên kết UUID Mach-O và dSYM đã mở vẫn chưa được quan sát. Tên biểu tượng được in bên trong báo cáo là các xác nhận được ghi lại và không được xác minh. Các core ELF của Linux sử dụng quy trình làm việc [sự cố đã ghi lại](recorded-crashes.md) riêng biệt.

## Giới hạn và lỗi

Tệp được chọn bị giới hạn ở 1 MiB theo mặc định (tối đa 16 MiB). Tối đa 256 luồng, 8.192 khung, 1.024 hàng `usedImages`, và 16.384 khe cắm thanh ghi `threadState` được kiểm tra theo mặc định (tối đa 4096 / 65536 / 4096 / 262144). Ngân sách được kiểm tra trước khi phân tích cú pháp từng hàng, và các thành viên thân mà báo cáo không diễn giải sẽ được kiểm tra cú pháp mà không được giữ lại. Các khoản phụ cấp cố định giới hạn mức độ lồng nhau của JSON (64 cấp), tên khóa không được liệt kê cho mỗi đối tượng (1.024), và các mục `termination.reasons` (256).
