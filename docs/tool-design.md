# Thiết kế công cụ MCP (MCP Tool Design)

## Quyết định khi nào cần thêm một công cụ

Thiết kế xoay quanh câu hỏi của nhà phân tích. Kiểm tra các hợp đồng chuẩn hiện tại, khả năng của provider và công cụ hiện có gần nhất trước khi thay đổi bề mặt. Mở rộng một công cụ khi ý định và hợp đồng kết quả khớp nhau; thêm một công cụ cho một kết quả riêng biệt hoặc thẩm quyền khác biệt về mặt thực chất. Tên và ngữ nghĩa công khai duy trì tính trung lập với provider, với độ bao phủ chính xác được báo cáo theo từng provider.

## Chọn hình dạng công cụ

| Hình dạng             | Sử dụng cho                                 | Kết quả                                                                   |
| --------------------- | ------------------------------------------- | ------------------------------------------------------------------------- |
| `inspect`             | Dữ kiện về một mục tiêu, đối tượng, tài nguyên | Các trường liên quan, vị trí và tính khả dụng của facet                  |
| `search` / `list`     | Khám phá ứng viên                           | Thứ tự ổn định và ngữ cảnh; phân trang khi truy vấn hoặc định dạng cần   |
| `trace`               | Mối quan hệ giữa các bằng chứng             | Các cạnh có kiểu, dữ kiện hỗ trợ và các đường dẫn chưa giải quyết         |
| `compare`             | Các artifact hoặc quan sát được ghép đôi rõ ràng | Danh tính, độ bao phủ có thể so sánh và các delta được hỗ trợ            |
| `workflow`            | Một kết quả định kỳ cần kết hợp             | Câu trả lời nội dòng, bằng chứng đóng góp và các facet một phần           |
| `observe` / `capture` | Hành vi runtime                             | Thẩm quyền, hiệu ứng khởi chạy/đính kèm, vòng đời và trạng thái dọn dẹp  |

## Ưu tiên các nguyên mẫu (Primitives); kết hợp quy trình làm việc (Workflows)

Một primitive báo cáo một dữ kiện có thể tái sử dụng về một đối tượng hoặc mối quan hệ đã được xác định. Bộ giải mã đặc thù theo định dạng đủ điều kiện; diễn giải nghiệp vụ của một ứng dụng thuộc về bên ngoài hợp đồng chung. Chủ sở hữu định dạng báo cáo các quan sát nguồn và xác thực cấu trúc; chủ sở hữu ứng dụng hoặc quy trình áp dụng chính sách tiếp nhận đặc thù theo trường hợp sử dụng mà không diễn giải lại các trường thô.
