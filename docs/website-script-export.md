# Xuất script website đã thu thập và phân tích JavaScript tĩnh

`export_web_scripts` kết nối các nguồn trình duyệt được giữ lại vào bộ phân tích JavaScript cục bộ hiện có. Cả CLI và MCP đều sử dụng cùng một quy trình làm việc. Nó đọc bản thu thập đã lưu từ `inspect_web_page` hoặc `capture_browser_scenario` và ghi các script cùng một manifest đã xác minh vào một thư mục chưa tồn tại do bên gọi chọn.

## Thu thập và xuất

Để kiểm tra thụ động, chọn `include_script_sources: true` (CLI: `--include-script-sources`). Đối với các kịch bản, chọn `capture.network.response_body: true`. Lưu bản ghi Bằng chứng hoàn chỉnh hoặc kết quả chuẩn hóa của nó dưới dạng UTF-8 JSON. `structuredContent` của MCP chứa bản ghi Bằng chứng hoàn chỉnh.

Bản thu thập phải là một tệp thông thường. Một liên kết mềm đến tệp thông thường được chấp nhận; thư mục, pipe có tên và tệp thiết bị bị từ chối trước khi đọc byte thu thập hoặc tạo đầu ra.

```sh
rea export-web-scripts /analysis/capture.json /analysis/exported-scripts --json
rea analyze-javascript-application /analysis/exported-scripts/files --json
```

MCP sử dụng `export_web_scripts` với đường dẫn hệ thống tệp tuyệt đối `capture_path` và `output_directory`. Kết quả nội dòng của nó bao gồm `analysis_input`, có thể được truyền trực tiếp đến `analyze_javascript_application`. Khi không có byte script nào có thể xuất, `analysis_input` là null và manifest giữ lại các nguồn không khả dụng cùng lý do của chúng. Hãy thu thập lại với tùy chọn nội dung tương ứng.

Thư mục đích phải chưa tồn tại. Sử dụng thư mục mới cho mỗi lần thu thập; một thư mục hoặc liên kết mềm hiện có sẽ bị từ chối mà không ghi đè lên nội dung của nó. REA tạo đầu ra riêng tư, xác minh mọi tệp được ghi bằng mã băm và đọc lại bền vững, đồng thời xóa đầu ra một phần của chính nó khi thất bại hoặc bị hủy bỏ. Một rollback thất bại sẽ báo cáo các đường dẫn còn sót lại.

## Nguồn và ánh xạ

Mỗi nguồn được trả về nội dòng cùng với URL của nó và danh tính script/frame hoặc tham chiếu chuỗi sự kiện và giao dịch yêu cầu. Kết quả cũng chứa mã băm byte của bản thu thập đầu vào, độ bao phủ ban đầu, liên kết Bằng chứng nguồn khi được cung cấp, cùng mã băm và số byte của manifest đã lưu trữ. Một phong bì Bằng chứng đã lưu phải có mã định danh ngữ nghĩa hợp lệ và thao tác khớp.

Các giá trị `relative_path` được xuất tương đối với `analysis_input.input_path`. Tệp `manifest.json` được lưu giữ chứa các trường manifest nội dòng tương tự, ngoại trừ bộ mô tả của chính nó. Nó không nhân bản các byte script.

- Nguồn thụ động là văn bản do Trình gỡ lỗi (Debugger) hiển thị được mã hóa dưới dạng UTF-8, với mã băm nguồn đã xác minh và khai báo source-map được giữ lại. Các nguồn không khả dụng hoặc rõ ràng không phải là JavaScript vẫn ở trạng thái không khả dụng.
- Nguồn kịch bản sử dụng các yêu cầu quan sát được phân loại là `script`, liên kết với một sự kiện phản hồi và các byte được trả về bởi API `Response.body()` của Playwright cho đối tượng phản hồi của nó. URL không bao giờ tự chọn phản hồi. Playwright có thể tải lại URL yêu cầu với thông tin xác thực khi CDP không có phần thân phản hồi, do đó các tham chiếu giao dịch và sự kiện không xác minh rằng các byte được xuất đến từ giao dịch mạng ban đầu. Bản thu thập và manifest xuất giữ lại giới hạn này. Các byte nhị phân và byte rỗng được giữ lại được ghi chính xác.
- Các đường dẫn URL `.js`, `.mjs`, và `.cjs` thông thường không có tham số truy vấn được sắp xếp dưới một thư mục cho mỗi origin, bảo toàn bố cục module tương đối của chúng khi nó không mơ hồ.
- Các phiên bản cạnh tranh, các biến thể truy vấn/đoạn, xung đột tiền tố tệp/thư mục, các nguồn nội dòng và các đường dẫn không thể biểu diễn sử dụng các tên cô lập duy nhất kèm theo lý do đã nêu. Các nguồn cạnh tranh bị thiếu cũng ngăn cản việc chọn một phiên bản dường như là chuẩn.
- Các cách viết URL phân biệt chữ hoa chữ thường giữ các đường dẫn tương đối của chúng khi hệ thống tệp đích có thể lưu trữ cả hai. Nếu một cách viết sau xung đột khi xuất bản, bản ghi đó sẽ bị cô lập trong khi cách viết chuẩn trước đó có thể vẫn tồn tại. Đọc `layout`, `relative_path`, và `layout_reason` của từng bản ghi; việc cô lập phân biệt chữ hoa chữ thường phụ thuộc vào đích đến, không giống như việc cô lập đường dẫn/phiên bản chính xác được lên kế hoạch trước.

## Diễn giải và giới hạn

Việc xuất không tìm nạp URL hoặc thực thi JavaScript. Các nguồn Trình gỡ lỗi và byte phản hồi được giữ lại không xác lập việc thực thi hoặc một trang web hoàn chỉnh. Byte phản hồi kịch bản bảo toàn kết quả API body của Playwright; xem giới hạn danh tính body nguồn ở trên. Việc biên tập (redaction) đã khai báo được bảo toàn và có thể thay đổi cú pháp JavaScript; các bản thu thập thụ động không báo cáo cờ biên tập byte, do đó giá trị đó vẫn là unknown.

Trình phân tích hiện có kiểm kê và phân tích cú pháp các tệp được xuất thực tế. Nó có thể giải quyết các module tương đối thu thập thông thường, nhưng không triển khai import map của trình duyệt, tải URL tương đối gốc, import từ xa hoặc hành vi runtime. Bộ giải quyết cục bộ của nó có thể loại bỏ các truy vấn/đoạn import; các biến thể cô lập được giữ cách xa các đường dẫn chuẩn để ngăn chúng chọn một phiên bản quan sát khác. Các phụ thuộc bị thiếu vẫn là unknown. Source map không được tìm nạp hoặc tái tạo trong quá trình xuất.

Bằng chứng xuất mô tả một dẫn xuất từ bản thu thập đã lưu. Bằng chứng phân tích ứng dụng tiếp theo mô tả artifact được xuất cục bộ. Danh tính và thẩm quyền của chúng vẫn tách biệt.

## Xác minh

`npm run verify:browser:scripts` thực hiện thu thập chủ động, xuất nguồn chính xác, đọc lại manifest, các biến thể truy vấn và phân tích module tương đối thông qua trình duyệt thực, CLI và stdio MCP. Đặt `REA_BROWSER_EXECUTABLE` thành trình duyệt thuộc họ Chrome đã cài đặt. Bộ xác minh chấp nhận một đối số điểm vào REA đã cài đặt tùy chọn để xác minh gói. `npm run verify:browser` cũng bao gồm xuất `inspect_web_page` thụ động thông qua cả hai adapter công khai.
