# Độ bao phủ tái cấu trúc (Reconstruction Coverage)

Lệnh `rea evaluate-reconstruction-coverage coverage.json --json` đánh giá một workspace bao phủ nội dòng và `boundary_id` đã chọn của nó. MCP hiển thị cùng một đầu vào thông qua `evaluate_reconstruction_coverage`. Kết quả chứa trạng thái, lý do, các đầu dò khuyến nghị và tham chiếu Bằng chứng; bundle được cung cấp giữ lại các bản ghi hoàn chỉnh.

Các ID Bằng chứng và manifest của bundle phải được xác thực trước khi đánh giá. Một bản ghi hợp lệ vẫn có thể là bằng chứng chưa đủ: một bản tóm tắt bộ xác minh đạt chuẩn không thể nâng Bằng chứng suy diễn thành một quan sát phát lại (replay observation). Trường hợp đó trả về `unknown`, trong khi bản ghi bị thiếu hoặc bị giả mạo vẫn là lỗi đầu vào.

## Phạm vi hoàn thành hiệu dụng

Ranh giới đã chọn yêu cầu hợp nhất các ID bề mặt rõ ràng của nó, các bề mặt của mọi xác nhận (claim) bắt buộc, và mọi phụ thuộc đã khai báo có thể tiếp cận được từ các bề mặt đó. Các chu trình chỉ được duyệt một lần. Các phụ thuộc tiếp cận được bị thiếu vẫn hiển thị dưới dạng `dependency-surface-missing`; chúng giữ cho việc đóng (closure) chỉ hoàn thành một phần. Tóm tắt `required_surfaces` đếm tập hợp đầy đủ này, bao gồm cả các ID vắng mặt, và `required_claims` đếm các xác nhận bắt buộc duy nhất.

Mỗi bề mặt tiếp cận được đều nhận cùng các kiểm tra về quyền sở hữu, đường dẫn, phân phối, bố trí và tính độc lập quyền hạn. Các mâu thuẫn đang hoạt động trên bất kỳ bề mặt nào trong số này đều ảnh hưởng đến ranh giới ngay cả khi bên gọi không lặp lại ID bề mặt trong `required_surface_ids`. Bằng chứng gói (Package proof) phải bao gồm các artifact của phạm vi hiệu dụng này. Bộ xác minh xác nhận phải cam kết với các bề mặt của chính nó và các phụ thuộc tiếp cận được của chúng, bao gồm cả mã băm artifact và chủ sở hữu đã triển khai của chúng. Bên gọi không cần sao chép bao đóng bắc cầu trong yêu cầu.

## Bằng chứng bộ xác minh có thể so sánh

Ít nhất một bản ghi được trích dẫn rõ ràng bởi quan sát bộ xác minh mới nhất phải hỗ trợ thẩm quyền hợp đồng đã khai báo:

| Thẩm quyền hợp đồng | Nguồn Bằng chứng (Evidence source)        | Độ tin cậy (Confidence) |
| ------------------- | ----------------------------------------- | ----------------------- |
| `shipped-artifact`  | `shipped-artifact`                        | `observed` hoặc `derived` |
| `controlled-replay` | `controlled-replay`                       | `observed`              |
| `live-observation`  | `controlled-replay` hoặc `external-service` | `observed`              |
| `external`          | `external-service`                        | `observed`              |

Các tham chiếu theo ngữ cảnh có thể đi kèm bằng chứng đủ điều kiện. Một bản ghi quan sát được không liên quan ở nơi khác trong bundle không thể sửa chữa một kết quả chỉ trích dẫn suy diễn. Các kết quả quan sát được cũ hơn không thể thay thế kết quả yếu mới nhất. Các quan sát tương đương tại cùng một thời điểm có thể chia sẻ bằng chứng đủ điều kiện từ một trong những quan sát đó; các đánh giá mâu thuẫn ngang nhau vẫn giữ trạng thái unknown.

## Bằng chứng gói và tích hợp

Các xác nhận về bản dựng (build), cài đặt sạch (clean-install) và điều hướng runtime yêu cầu Bằng chứng runtime quan sát được (`controlled-replay` hoặc `external-service`). Việc kiểm toán nội dung gói và tính độc lập thẩm quyền cũng có thể sử dụng Bằng chứng `shipped-artifact` quan sát được hoặc dẫn xuất. Mỗi bằng chứng phải trích dẫn các bản ghi đủ điều kiện của nó và bao phủ các mã băm artifact của ranh giới đã chọn. Chỉ riêng suy diễn không xác lập quy trình làm việc của gói đạt hay không đạt.

Bộ đánh giá bảo toàn nguồn gốc Bằng chứng và trả về một unknown có thể hành động khi không có đủ điều kiện. Nó không thay đổi các bản ghi được trích dẫn để làm cho thẩm quyền của chúng khớp với khai báo của bên gọi.

Tất cả các kết quả bằng chứng gói áp dụng đều tham gia đánh giá. Một bằng chứng áp dụng thất bại sẽ làm thất bại việc đóng ngay cả khi một bằng chứng khác đạt; các kết quả áp dụng unknown, unsupported hoặc bị bỏ qua sẽ giữ nó ở trạng thái unknown. Kết quả trích dẫn tất cả các kết quả áp dụng. Thứ tự mảng bằng chứng không xác lập độ mới hay sự thay thế. Một bằng chứng cho các mã băm artifact không liên quan không mâu thuẫn với ranh giới đã chọn.

## Các xác nhận tái cấu trúc hàm

`verify_reconstruction` đánh giá các xác nhận đã khai báo trên Bằng chứng so sánh được lưu giữ. Một xác nhận hàm `overall` bao gồm các ẩn số (unknowns) tồn dư đang hoạt động thuộc phạm vi của bất kỳ chiều nào trong phép so sánh đó. Ví dụ, một câu hỏi mã giả chưa có lời giải giữ cho xác nhận tổng thể là `unknown` và giữ lại ID ẩn số cùng đầu dò khuyến nghị của nó; một xác nhận assembly độc lập vẫn có thể đạt. Một ẩn số chỉ thuộc phạm vi `overall` không chặn mọi chiều riêng lẻ.

Việc lựa chọn sử dụng bản sửa đổi mới nhất của từng ẩn số, danh tính so sánh và tham chiếu Bằng chứng của nó. Các phụ thuộc ẩn số rõ ràng vẫn có liên quan trên các chiều. Các ẩn số cho một phép so sánh khác và các điểm đầu đã giải quyết không trở thành yếu tố cản trở chỉ vì Bằng chứng nguồn của chúng trùng lặp.
