# Ứng dụng thực thi đơn lẻ Node.js (Node.js Single Executable Applications - SEA)

REA kiểm kê các blob Node.js SEA được tiêm vào thông qua `inspect-artifact` / `inspect_artifact` và hiện thực hóa nội dung của chúng thông qua `extract-artifact` / `extract_artifact`. Nó không bao giờ chạy ứng dụng hoặc giải tuần tự hóa các snapshot V8 hay code cache.

```sh
rea inspect-artifact ./application --json
rea inspect-artifact ./application --detail summary --evidence_output evidence.json --json
rea extract-artifact ./application --json
```

Đối với MCP, mở vật mang (carrier) bằng `open_binary`, sau đó gọi `inspect_artifact`. Phân phối tóm tắt giữ lại Bằng chứng hoàn chỉnh: sử dụng `parent_evidence_id` với `inspect_analysis_view` để yêu cầu các đường dẫn và trường kiểm kê cụ thể. Việc trích xuất trả về một `output_root` mới; phân tích thư mục đã trích xuất đó bằng `analyze_javascript_application`. Các ứng dụng chỉ có snapshot không có script chính được khôi phục và không nhận được đầu dò JavaScript tiếp theo. Script chính có đường dẫn logic thiếu phần mở rộng nguồn JavaScript, chẳng hạn như `main`, cũng không nhận được đầu dò, vì bộ phân tích chọn các tệp được trích xuất theo phần mở rộng.

## Nhận diện và bằng chứng

Định dạng container là `node-sea`. REA định vị các section Mach-O có tên `__NODE_SEA_BLOB` trong `NODE_SEA` hoặc `__NODE_SEA`, các note ELF thuộc sở hữu của `NODE_SEA_BLOB`, và tài nguyên PE RCDATA có tên `NODE_SEA_BLOB`. Vị trí tài nguyên PE đến từ trình phân tích tài nguyên PE dùng chung. Các phân đoạn program-header `PT_NOTE` của ELF có thẩm quyền, như lúc runtime; các section `SHT_NOTE` chỉ được đọc khi không thể đọc bảng program-header, vì các bộ tiêm có thể để lại các mục section cũ. Một tệp PE không có tên tài nguyên `NODE_SEA_BLOB` được để lại cho phân loại native. Các vật mang Mach-O phổ quát (universal) với các lát cắt blob khác nhau bị từ chối vì mơ hồ; các blob giống hệt nhau được kiểm kê một lần. Các byte vật mang native giữ mã băm nội dung và phân loại native trong khi lần xuất hiện gốc đại diện cho container SEA.

Metadata gốc `format_metadata` bảo toàn cờ thô, cờ đã giải mã, đường dẫn mã, vị trí blob, đối số thực thi (exec arguments) và các quan sát cầu chì (fuse). Các phần tử con chính và tài sản giữ vai trò và tên khai báo chính xác trong `source_path`. Các tên an toàn, không xung đột trở thành đường dẫn logic sau khi chuẩn hóa đường dẫn NFC dùng chung. Các tên không an toàn dưới dạng đường dẫn tương đối, có xung đột chính xác hoặc tiền tố tệp với mục khác, hoặc chiếm namespace dành riêng `__node_sea__` sẽ nhận các đường dẫn logic tất định: `__node_sea__/main.cjs` cho nguồn CommonJS chính và `__node_sea__/assets/<ordinal>` cho tài sản, sử dụng thứ tự tính từ 0 của chúng trong blob. Tên khai báo ban đầu vẫn còn trong `source_path`, bao gồm cả khóa tài sản trống. Snapshots và code cache sử dụng `__node_sea__/snapshot` và `__node_sea__/code-cache`. Việc trích xuất hiện thực hóa các đường dẫn logic thông qua các quy tắc ngăn chặn đường dẫn và xung đột dùng chung và không bao giờ ghi đè lên các tệp hiện có.

Quét cầu chì (fuse) ghi lại độc lập các trạng thái `set`, `unset`, `absent`, hoặc `conflicting`. Cầu chì chưa đặt với một blob hiện diện được báo cáo là một giới hạn. Đây là các quan sát lính gác (sentinel) tĩnh, không phải bằng chứng về việc kích hoạt lúc runtime: văn bản giống lính gác trong byte script hoặc tài sản cũng tham gia vào quá trình quét.

## Bố cục và giới hạn được hỗ trợ

Bộ giải mã tuân theo bố cục little-endian 64-bit trong [v24.10.0 node_sea.cc](https://github.com/nodejs/node/blob/v24.10.0/src/node_sea.cc) và [node_sea.h](https://github.com/nodejs/node/blob/v24.10.0/src/node_sea.h) của Node, bao gồm cả byte mở rộng đối số thực thi. Các bố cục cũ hơn không có byte đó, các trường độ dài 32-bit, các blob big-endian và các cờ trong tương lai không được hỗ trợ. Thứ tự byte của vật mang độc lập với thứ tự byte của blob.

Magic bị lỗi, bị cắt cụt, cờ/phần mở rộng không xác định, UTF-8 không hợp lệ, khóa tài sản thô trùng lặp và các khai báo blob xung đột sẽ thất bại đóng với lý do được nêu tên. Phần đệm số 0 sau một blob hoàn chỉnh được cho phép. Mức trần đọc blob là 256 MiB; vật mang PE cũng sử dụng các mức trần đầu vào 512 MiB, 65.536 mục thư mục và 8 MiB metadata tài nguyên dùng chung. Bộ giải mã tiếp nhận tối đa 100.000 tài sản có tổng tên tối đa 8 MiB, và 4.096 đối số thực thi với tổng cộng tối đa 1 MiB văn bản. Đây là các giới hạn tài nguyên cố định, không phải giới hạn định dạng của Node, được kiểm tra trước khi giải mã tên hoặc tạo bản ghi; chạm giới hạn sẽ thất bại mà không có ghi đè runtime. Các byte snapshot và code-cache vẫn ở dạng mờ đục (opaque).

## Xác minh

Các fixture tổng hợp bao gồm kiểm kê/trích xuất Mach-O, ELF và PE, các blob lỗi, xung đột và mâu thuẫn cầu chì. Các kiểm tra CLI và MCP thực hiện cùng một artifact provider và các schema được quảng bá. Hồi quy CLI cũng tạo một blob với phiên bản Node cục bộ được ghim, đường dẫn chính tuyệt đối và khóa tài sản trống, sau đó kiểm kê nó bên trong một vật mang ELF tổng hợp. Điều này xác minh trình tạo blob thực tế, không phải tệp thực thi được tiêm thực tế.
