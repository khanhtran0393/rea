# Lời nhắc MCP và gợi ý hoàn thành có hướng dẫn (Guided Prompts & Completion)

REA cung cấp sáu quy trình điều tra trung lập với provider thông qua tính năng `prompts` của MCP. Chúng bổ trợ cho danh mục công cụ được tạo tự động; bản thân chúng không tự ý thêm, xóa hoặc gọi công cụ.

Các client hỗ trợ công cụ nhưng không hỗ trợ prompts có thể sử dụng công cụ không phụ thuộc mục tiêu `guided_workflows` với `action: "list"` hoặc `action: "render"`. Người dùng CLI có thể dùng `rea guided-workflows` để liệt kê các workflow, hoặc truyền `--action render`, `--workflow-id ID`, và `--arguments-json '{"feature":"offline search"}'` để hiển thị một workflow. Cả hai cách đều sử dụng cùng các hợp đồng và bộ hiển thị như lời nhắc native; việc hiển thị chỉ trả về các chỉ dẫn hướng dẫn và không kiểm tra mục tiêu, không bắt đầu phân tích hay thay đổi trạng thái phiên làm việc.

Danh sách chứa định danh quy trình, mô tả và các yêu cầu đối số; lệnh render chỉ trả về nội dung văn bản của quy trình đã chọn. Tên đối số không xác định và giá trị không phải chuỗi sẽ bị từ chối kèm đường dẫn trường lỗi. `action: "list"` không nhận định danh quy trình hay đối số. Bộ điều hợp MCP tuân thủ cùng phạm vi `REA_MCP_TOOL_FAMILIES` như lời nhắc native, bao gồm cả yêu cầu render.

Để tìm một phần trong gói hướng dẫn đã cài đặt theo câu hỏi của chuyên viên phân tích, sử dụng `search_guidance` và truyền bộ định vị (locator) trả về vào `read_guidance_section`. Lệnh CLI tương đương là `rea guidance --query "exact JavaScript module"`, tiếp theo là `rea guidance --action read --locator LOCATOR`. Tìm kiếm chỉ giới hạn trong các tệp skill được quản lý đóng gói sẵn, hoạt động không cần provider hay mục tiêu, và trả về các kết quả khớp được phân trang xác định. Mọi từ truy vấn không phổ quát phải xuất hiện trong phần đó; truy vấn chỉ gồm từ phổ quát sẽ không trả về kết quả. Kết quả là tài liệu; các locator được liên kết với gói đã cài đặt và báo cáo hướng dẫn cũ hoặc không khả dụng mà không trả về nội dung phần được lưu trong bộ nhớ đệm. Kết quả không khớp không nói lên điều gì về tính khả dụng của năng lực phân tích.

Tìm kiếm trả về tối đa 20 kết quả khớp mỗi trang kèm đoạn trích ngữ cảnh 500 đơn vị mã. Lệnh Read truy xuất toàn bộ phần đã chọn. Tiếp nhận cho phép 1 MiB mỗi tệp, 4 MiB mỗi gói, 4.096 phần và 25.165.824 đơn vị mã UTF-16 mở rộng trước khi văn bản phần được cụ thể hóa. Các gói không đọc được hoặc quá kích thước sẽ trả về `bundle_unavailable`, không có chỉ mục một phần hoặc suy diễn sự vắng mặt. Lỗi dọn dẹp vẫn là lỗi kèm chủ sở hữu thử lại được lưu giữ. Gói và phần hoàn chỉnh được ràng buộc bởi các mã băm kích thước cố định trong locator, do đó tiêu đề dài không làm cho locator của chính nó trở nên không đọc được.

## Danh mục lời nhắc (Prompt Inventory)

| Lời nhắc | Mục đích |
| :--- | :--- |
| `investigate_feature` | Lần vết một tính năng từ phát hiện ban đầu đến bằng chứng hàm (function evidence) liên quan. |
| `compare_application_versions` | So sánh hai hiện vật phát hành, với các bước theo dõi tĩnh và runtime tùy chọn. |
| `verify_reconstruction` | Đánh giá một đặc tả khôi phục hữu hạn dựa trên Bằng chứng (Evidence) so sánh được lưu giữ. |
| `trace_crash` | Tương quan triệu chứng sự cố (crash) với các đường dẫn tĩnh và bằng chứng bắt giữ tiến trình tùy chọn. |
| `audit_residual_unknowns` | Kiểm toán các điểm chưa xác định còn sót lại (residual-unknown) và độ phân giải đủ điều kiện bằng chứng. |
| `prepare_bounded_process_capture` | Thiết kế thử nghiệm tiến trình có giới hạn và kiểm soát trước khi thực thi. |

Phạm vi `REA_MCP_TOOL_FAMILIES` chỉ quảng bá một quy trình làm việc khi mọi công cụ trong các bước đề xuất của nó thuộc về một nhóm được chọn. Các quy trình làm việc khả dụng một phần sẽ bị loại bỏ khỏi `prompts/list` — `prompts/get` báo cáo chúng là không tìm thấy — nhờ đó lời nhắc được render không bao giờ hướng dẫn tác tử gọi lệnh bị vô hiệu hóa. Mã nhận diện server và danh mục lời nhắc của `rea mcp doctor` tuân theo cùng một phạm vi.

Mỗi lời nhắc được render cung cấp các điểm bắt đầu tùy chọn bằng tên công cụ REA hiện tại. Các tác tử có thể gọi trực tiếp công cụ, bỏ qua các đề xuất không liên quan và chỉ kiểm tra trạng thái provider hoặc mục tiêu khi điều đó giúp trả lời yêu cầu hoặc công cụ đòi hỏi. Lời nhắc cũng yêu cầu tác tử phân biệt rõ ràng giữa quan sát (observations), suy luận (inferences) và điểm chưa xác định (unknowns). Các đối số lời nhắc được yêu cầu và lựa chọn hoàn thành được render dưới dạng dữ liệu lựa chọn không tin cậy, không phải là hướng dẫn hay sự ủy quyền. Kết quả công cụ bao gồm Bằng chứng (Evidence) nội tuyến. Lời nhắc không yêu cầu tìm nạp gói hoặc tài nguyên để đọc kết quả; ID Bằng chứng chỉ là tham chiếu khi đối số lời nhắc hoặc công cụ chấp nhận rõ ràng.

Đối với thư mục ứng dụng JavaScript/Electron, `investigate_feature` trỏ trực tiếp đến `analyze_javascript_application` với `input_path`; nó không yêu cầu `open_binary`. Khi Bằng chứng đó, hoặc Bằng chứng `inspect_binary_layout`, đã được lưu giữ, nó đề xuất `inspect_analysis_view` trước khi lần vết hạt giống tính năng. Lệnh CLI tương đương là `rea analyze <thư_mục>`.
`open_binary` và lệnh kiểm tra mục tiêu của doctor chấp nhận các tệp và thư mục gói Apple (`.app`, `.appex`, `.framework`, `.xpc`, `.systemextension`, `.bundle` và `.plugin`). Loại gói Apple khác bị từ chối là không được hỗ trợ, và thư mục khác được báo cáo kèm một hành động phân tích khả dụng; điều này không xác lập sự hỗ trợ cho mọi định dạng thư mục.

Đối với đường dẫn mục tiêu trong `compare_application_versions`, lời nhắc đề xuất mở từng mục tiêu trước khi gọi `inspect_artifact`; việc kiểm tra hoạt động trên mục tiêu đang hoạt động và trả về đồ thị cùng phát hiện nội tuyến.

Sử dụng khám phá và truy xuất MCP tiêu chuẩn:

```json
{ "method": "prompts/list", "params": {} }
```

```json
{
  "method": "prompts/get",
  "params": {
    "name": "investigate_feature",
    "arguments": {
      "feature": "offline search",
      "document": "Notes"
    }
  }
}
```

Server thông báo `listChanged: true`. Việc cập nhật lời nhắc có hướng dẫn đã đăng ký sẽ phát ra `notifications/prompts/list_changed`, cho phép client làm mới danh mục lời nhắc được lưu trong bộ nhớ đệm.

Các tham số không hợp lệ theo lược đồ gửi tới `prompts/get`, `prompts/list`, `completion/complete`, và `tools/list` trả về JSON-RPC `-32602` kèm chẩn đoán trường. Hãy sửa các trường được nêu tên và thử lại trên cùng một kết nối. Các lỗi xử lý không mong đợi trả về JSON-RPC `-32603`.

## Gợi ý hoàn thành nhận biết phiên (Session-Aware Completion)

`completion/complete` của MCP hoàn thành các đối số lời nhắc có hướng dẫn; nó không định nghĩa tính năng hoàn thành cho đối số gọi công cụ tùy ý. REA gắn gợi ý hoàn thành vào các đối số lời nhắc tùy chọn chấp nhận giá trị theo phiên. Gợi ý chỉ cung cấp định danh; chúng không truy xuất hoặc thay thế kết quả Bằng chứng nội tuyến.

```json
{
  "method": "completion/complete",
  "params": {
    "ref": {
      "type": "ref/prompt",
      "name": "investigate_feature"
    },
    "argument": {
      "name": "procedure",
      "value": "0x10"
    },
    "context": {
      "arguments": {
        "document": "Notes"
      }
    }
  }
}
```

Nguồn hoàn thành là hình chiếu trực tiếp của phiên hiện tại:

Gợi ý tài liệu và thủ tục (procedure) trong phiên nhị phân đọc provider đã sẵn sàng; nó không bắt đầu phân tích. Một provider chưa tải (cold), chuyển đổi mục tiêu, hoặc danh mục kiểm kê không khả dụng có thể không trả về gợi ý nào. Hãy chạy truy vấn phân tích trước khi bạn cần kiểm kê từ provider cung cấp dữ liệu đó. Danh sách hoàn thành rỗng không chứng minh mục tiêu không có thủ tục.

| Nhóm đối số | Nguồn và bộ lọc |
| :--- | :--- |
| Document | Kết quả `list_documents` của provider hiện tại. |
| Procedure | Toàn bộ kiểm kê `list_procedures`; tất cả địa chỉ thủ tục và thủ tục có tên duy nhất đều được đưa ra. Ngữ cảnh tài liệu tùy chọn được chuyển tiếp. |
| Provider | `auto` cộng với các ứng viên chuyên sâu đã sắp xếp từ phiên nhị phân hiện tại; gợi ý không phụ thuộc mục tiêu không khởi động chúng. |
| Evidence | ID Bằng chứng được lưu giữ trong sổ cái phiên hiện tại, chỉ cung cấp cho các đối số lời nhắc chấp nhận tham chiếu Bằng chứng. |
| Process capture | ID cho Bằng chứng được lưu giữ có thao tác và kết quả xác thực xác định việc bắt giữ tiến trình. |
| Artifact manifest | ID Manifest từ các kiểm kê hiện vật được lưu giữ hợp lệ theo lược đồ. |
| Artifact occurrence | ID Occurrence có mặt trong các trang kiểm kê hiện vật được lưu giữ hợp lệ theo lược đồ. |
| Residual unknown | Chỉ các điểm chưa xác định hiện chưa được giải quyết. |

Khớp tiền tố được chuẩn hóa Unicode và không phân biệt chữ hoa chữ thường; các đề xuất giữ nguyên cách viết gốc. Server loại bỏ trùng lặp và sắp xếp theo điểm mã (code point) để cho kết quả xác định. Gợi ý thủ tục lấy ứng viên từ toàn bộ danh mục kiểm kê, do đó một tên chỉ được đưa ra khi nó xác định duy nhất một địa chỉ trong danh mục đó. Mỗi tên tại một địa chỉ dùng chung đều tham gia vào việc kiểm tra này, bao gồm cả các hàng sau xác lập tính mơ hồ. Phản hồi gợi ý mang tối đa 100 giá trị (giới hạn MCP). `total` đếm tất cả kết quả khớp và `hasMore` là `true` khi có giá trị bị bỏ sót. Hoàn thành không có điều khiển phân trang. Tiền tố dài hơn chỉ tiếp cận giá trị bị bỏ sót khi nó loại trừ các kết quả khớp khác; việc khớp sử dụng dạng chuẩn hóa, không phân biệt hoa thường, do đó một giá trị có thể không tiếp cận được khi có hơn 100 kết quả khớp dùng chung tiền tố chuẩn hóa của nó.

## Vòng đời và an toàn (Lifecycle & Safety)

Gợi ý hoàn thành không có bộ nhớ đệm trong REA. Việc mở hoặc chuyển đổi mục tiêu sẽ thay đổi đề xuất tài liệu và thủ tục trong yêu cầu tiếp theo. Đóng tệp nhị phân sẽ xóa sổ cái Bằng chứng và sổ đăng ký residual-unknown của nó, do đó các định danh từ phiên đã đóng sẽ không còn được đề xuất nữa. Lỗi provider, thao tác không được hỗ trợ và đầu ra provider bị sai định dạng tạo ra danh sách hoàn thành rỗng thay vì một định danh chưa được xác minh.

Gợi ý là điểm bắt đầu, không phải là lệnh gọi công cụ. Một công cụ tác động lên các trường mục tiêu, thao tác và vòng đời trong yêu cầu của chính nó; việc chọn một gợi ý không thực thi nó hoặc mở rộng các trường đó.
