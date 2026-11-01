# Phân tích tĩnh trong GitHub Actions

Quy trình làm việc [mẫu tối giản](https://github.com/morluto/rea/blob/main/examples/github-actions/rea-analysis.yml) xây dựng một ứng dụng JavaScript nhỏ, phân tích các tệp của nó mà không cần thực thi chúng, và tải lên các kết quả mà máy đọc được. Nó sử dụng CLI hiện có, không phải GitHub Action riêng biệt hay MCP client. Nó không đảm bảo hỗ trợ cho mọi provider của REA.

## Chạy ví dụ

1. Sao chép ví dụ vào `.github/workflows/rea-analysis.yml` trong kho lưu trữ của bạn và commit nó vào nhánh mặc định. Không cần ứng dụng hay manifest gói hiện có cho bản trình diễn độc lập này.
2. Bật GitHub Actions nếu cần. Chọn **REA static analysis example** trong tab Actions và chọn **Run workflow**.
3. Tải xuống artifact `rea-analysis` sau khi chạy xong. Các tệp của nó bao gồm:

   | Tệp               | Ý nghĩa                                                                                             |
   | ----------------- | --------------------------------------------------------------------------------------------------- |
   | `analysis.json`   | CLI JSON gốc, bao gồm Bằng chứng hoàn chỉnh khi thành công hoặc báo cáo lỗi khi có lỗi phát sinh.  |
   | `analysis.stderr` | Chẩn đoán/tiến trình gốc; tệp trống là bình thường khi không có cảnh báo.                          |
   | `exit-code.txt`   | Mã thoát (exit code) gốc của tiến trình phân tích.                                                 |

Ví dụ ghim Node.js 24.22.1, REA 6.7.0, và các bản sửa đổi commit của action. Nó cài đặt REA vào thư mục tạm thời của runner thay vì thay đổi các phụ thuộc của ứng dụng. Điều này ghim phiên bản công cụ, không phải mọi phụ thuộc npm bắc cầu; hãy sử dụng lockfile gói đã được xem xét, cam kết và `npm ci` nếu tích hợp của bạn yêu cầu bảo đảm khả năng tái tạo phụ thuộc mạnh mẽ hơn.

## Điều kiện tiên quyết và sự tin cậy

Runner là Ubuntu 24.04 do GitHub lưu trữ với Bash, npm, và `ps` từ `procps`. Các hình ảnh Linux khác phải cung cấp `procps`; các điều khiển tiến trình được giám sát của REA sử dụng nó. Thao tác JavaScript tĩnh này không cần Hopper, Ghidra, IDA, Java, trình duyệt, Docker, GPU, khóa API bên ngoài hay secret GitHub do người dùng cung cấp. Quy trình làm việc chỉ yêu cầu quyền `contents: read`; việc tải lên artifact sử dụng thông tin xác thực Actions bình thường của runner.

Thay thế **Build a known artifact** bằng bản dựng đã được kiểm duyệt của bạn, và đặt `ARTIFACT_PATH` thành thư mục ứng dụng của nó (tương đối với bản checkout) hoặc một tệp ASAR. Bản trình diễn tự tạo ứng dụng nhỏ của riêng nó; nó cố tình văng lỗi (throw) nếu được thực thi. REA chỉ đọc nó một cách tĩnh. Bản dựng thay thế của bạn có thể tự thực thi mã: hãy kiểm duyệt nó riêng và không thêm các bản dựng pull-request không đáng tin cậy vào quy trình làm việc có đặc quyền `pull_request_target`. Ví dụ được kích hoạt thủ công, không phải là chính sách thực thi PR không đáng tin cậy.

Các báo cáo có thể chứa đường dẫn, URL, dữ kiện dẫn xuất từ nguồn và dữ liệu do bên gọi chọn. Chỉ phân tích các artifact và xuất bản các báo cáo mà bạn được ủy quyền chia sẻ. Không dựa vào việc REA suy đoán các secret từ tên của chúng. Xem xét quyền truy cập và thời gian lưu giữ artifact cho kho lưu trữ của bạn; ví dụ lưu giữ các báo cáo trong 7 ngày.

## Thất bại, kết quả một phần và năng lực không được hỗ trợ

Bước phân tích thu thập riêng biệt stdout và stderr, ghi lại `$?`, và thoát với cùng trạng thái đó. Không có `continue-on-error`, `|| true`, bộ lọc đầu ra hay pipeline nào có thể biến một phân tích thất bại thành trạng thái thành công (green). Trước khi checkout hoặc các bước có thể xảy ra lỗi khác, quy trình làm việc tạo một thư mục báo cáo mới dưới `RUNNER_TEMP`. Phân tích và tải lên đều sử dụng thư mục đó, không bao giờ dùng `reports/rea` đã commit hoặc các tệp workspace khác. Bước tải lên sử dụng `always()` khi thư mục đó đã được chuẩn bị, do đó các chẩn đoán có sẵn vẫn tồn tại sau một phân tích thất bại. Nếu quá trình cài đặt hoặc xây dựng thất bại trước khi phân tích bắt đầu, thư mục sẽ trống và bước tải lên sẽ cảnh báo thay vì xuất bản kết quả cũ hoặc tạo ra một kết quả thành công giả. Việc mất runner hoặc hủy bỏ vẫn có thể ngăn cản việc tải lên.

`--json --full-output` chọn phong bì mà máy đọc được; nó **không** có nghĩa là phân tích đã hoàn tất. Một thao tác thành công có thể giữ lại phân tích một phần, các ẩn số và giới hạn. Thoát với mã 0 không phải là phán quyết bảo mật, bằng chứng về độ bao phủ đầy đủ hay cổng chính sách cho các phát hiện đó. Hãy kiểm tra Bằng chứng và độ bao phủ đã lưu trước khi đưa ra quyết định như vậy. Xem [đầu ra và trạng thái thoát](cli.md#output-and-exit-status).

Một mã thoát khác 0 có thể đại diện cho đầu vào không hợp lệ, công cụ không khả dụng, máy chủ/năng lực không được hỗ trợ hoặc một lỗi thao tác khác. Giữ lại lỗi JSON gốc (khi được phát ra), stderr, và mã thoát, đồng thời sử dụng danh mục/thông báo của chúng để chẩn đoán. Một lỗi khởi động sớm có thể để lại `analysis.json` trống; không phân tích cú pháp đó dưới dạng phân tích thành công. Chọn một thao tác được hỗ trợ hoặc cung cấp rõ ràng các [điều kiện tiên quyết đã được ghi chép](installation.md); không âm thầm chuyển đổi provider, thực thi artifact hoặc bỏ qua lỗi để có được một job màu xanh.
