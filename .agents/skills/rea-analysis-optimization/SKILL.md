---
name: rea-analysis-optimization
description: Tối ưu hóa các provider và quy trình phân tích dùng chung của REA về ngữ cảnh tác tử, độ trễ, CPU và bộ nhớ trong khi vẫn bảo toàn tính chính xác và Evidence. Sử dụng cho các điểm nghẽn phân tích và thiết kế bộ nhớ đệm hoặc truy vấn; tối ưu hóa bộ kiểm thử/CI thuộc về rea-ci-tests-audit.
---

# Tối ưu hóa phân tích REA (REA analysis optimization)

Tìm nguồn phát sinh công việc không cần thiết cụ thể, thay đổi nó tại chủ thể sở hữu và xác minh các cuộc điều tra hoàn chỉnh tương đương. Một kết quả hữu ích cũng có thể là quyết định có đo lường về việc giữ nguyên một triển khai hiện có.

Tuân theo [các ưu tiên hiệu năng](../../../AGENTS.md#performance), [thiết kế công cụ](../../../docs/tool-design.md) và [hướng dẫn kiểm thử](../../../docs/testing.md) của bản checkout. Kỹ năng này cung cấp một quy trình tối ưu hóa, không phải cấp thêm quyền thực thi mục tiêu, xuất bản hoặc merge. Sử dụng các hợp đồng hiện tại, điều kiện tiên quyết của provider và ủy quyền của người dùng.

Dành riêng việc đo lường hiệu năng cho các thay đổi nhắm vào hiệu năng hoặc có lo ngại cụ thể về suy giảm hiệu năng (regression). Dọn dẹp định kỳ sử dụng các kiểm tra tính đúng đắn và không yêu cầu quy trình tối ưu hóa này hoặc các benchmark tổng hợp chỉ vì các lệnh gọi helper, import hoặc biểu diễn thay đổi.

Đọc [nghiên cứu điển hình Kuna](references/kuna-case-study.md) khi chọn chiến lược tái sử dụng hoặc diễn giải một benchmark. Các quan sát được ghim của nó minh họa cho các sự đánh đổi; chúng không phải là bảo đảm của provider hiện tại hoặc sự tăng tốc phổ quát.

## Ghim câu hỏi và chuẩn đối sánh tính đúng đắn (Pin the question and the correctness oracle)

Mô tả tác vụ đã hoàn thành của nhà phân tích, bao gồm các dữ kiện và mối quan hệ cần thiết để trả lời câu hỏi đó, chi tiết được yêu cầu, quyền truy cập nguồn và các điểm chưa xác định có ý nghĩa. Chọn một hiện vật thực tế đại diện và một trường hợp nhỏ hoặc rỗng. Ghi lại phiên bản gốc (baseline revision), mã băm mục tiêu, danh tính engine, tùy chọn, máy chủ và điểm vào. Sử dụng một bản checkout riêng cho các thay đổi mà không làm ảnh hưởng đến công việc khác.

Thiết lập một chuẩn đối sánh (oracle) độc lập với triển khai đã thay đổi: kỳ vọng fixture được xây dựng từ nguồn, các byte tệp được ánh xạ độc lập hoặc một producer native ghép cặp với các kiểm tra về cách diễn giải của nó. So sánh hai lớp bọc xung quanh cùng một bộ giải mã lỗi không chứng minh được tính đúng đắn. Bảo toàn độ rộng địa chỉ, các giải pháp thay thế cùng địa chỉ, các điểm cuối chưa giải quyết, độ bao phủ và liên kết nguồn. Kết quả rỗng chỉ chứng minh sự vắng mặt trong phạm vi thực tế được kiểm tra.

## Xác định vị trí công việc không cần thiết sớm nhất (Locate the earliest unnecessary work)

Lần vết một tác vụ CLI hoặc MCP công khai thông qua mã ứng dụng dùng chung tới producer của nó. Đo lường các giai đoạn liên quan: kiểm tra danh tính và đầu vào, phân tích native, giải mã, tra cứu, chiếu (projection), tuần tự hóa, lưu giữ Evidence và dọn dẹp. Bao gồm khám phá danh mục và các lệnh gọi bổ sung khi chúng ảnh hưởng đến tác vụ đã hoàn thành.

Sử dụng bằng chứng để chọn sự hiệu chỉnh cấu trúc nhỏ nhất:

| Chi phí quan sát được | Đề xuất khắc phục | Điều kiện cần xác lập |
| :--- | :--- | :--- |
| Quét lặp lại các dữ kiện đã xác thực | Tái sử dụng cấu trúc tra cứu hoặc thứ tự hiện có | Chủ thể xác lập thứ tự và danh tính; mọi lựa chọn thay thế phù hợp đều được bảo toàn |
| Công việc engine tương đương bị lặp lại | Chia sẻ kết quả đã xác thực hoặc producer đang chạy | Đầu vào, độ tươi mới, vòng đời và kết quả cho mỗi bên gọi vẫn rõ ràng |
| Báo cáo đầy đủ cho các dữ kiện được chọn | Đẩy việc chọn lọc hoặc chiếu sang chủ thể có khả năng sớm nhất | Các dữ kiện và bằng chứng bắt buộc vẫn có thể truy cập được; độ bao phủ đã kiểm tra được báo cáo |
| Bộ đệm truyền tải tạm thời quá lớn | Giải mã tăng dần hoặc sử dụng các lần đọc nguồn có giới hạn | Xác thực toàn bộ tài liệu được chấp nhận; các dữ kiện và bản sao được lưu giữ vẫn tiêu tốn bộ nhớ |
| Nhiều lần đọc đồng nhất lặp đi lặp lại | Chia sẻ việc tiếp nhận, giải mã hoặc xử lý hàng loạt | Bảo toàn danh tính mục, thứ tự khi được yêu cầu và các lỗi riêng lẻ |

Không đưa vào một chỉ mục, bộ nhớ đệm chung, endpoint hàng loạt hoặc lớp điều phối mà không xác định công việc nó loại bỏ và chi phí duy trì/tài nguyên của nó. Việc chiếu sau khi hiện thực hóa một báo cáo đầy đủ có thể làm giảm đầu ra trong khi vẫn giữ nguyên thời gian phân tích và bộ nhớ đỉnh. Sửa chữa các dữ kiện bị thiếu hoặc không chính xác trước khi benchmark một bố cục thay thế so với một quy trình có độ bao phủ tốt hơn.

## Giữ một cách diễn giải duy nhất và vòng đời tái sử dụng xác định (Keep one interpretation and a defined reuse lifetime)

Để ranh giới producer xác thực và chuẩn hóa các dữ kiện một lần duy nhất. Các consumer truy vấn biểu diễn đó thông qua triển khai CLI/MCP dùng chung; adapter không nên xây dựng lại các mối quan hệ hoặc độc lập diễn giải lại các báo cáo thô. Giữ danh tính tra cứu tách biệt khỏi định dạng hiển thị. Tái sử dụng hợp đồng khi mục đích của chúng khớp nhau, tôn trọng chi tiết được yêu cầu và quyền truy cập Evidence đầy đủ.

Đối với mỗi bộ nhớ đệm hoặc producer dùng chung, hãy nêu rõ khóa (key), chủ sở hữu, cơ chế vô hiệu hóa (invalidation), kích thước lưu giữ và vòng đời. Giải trình cho các byte mục tiêu ảnh hưởng đến kết quả, nội dung engine, tùy chọn phân tích, ánh xạ kiến trúc/tải và phiên bản trình phân tích cú pháp/lược đồ khi chúng ảnh hưởng đến việc tái sử dụng. Xác thực trạng thái trực tiếp trước khi gọi một quan sát được lưu trong bộ nhớ đệm là hiện hành. Chỉ lưu vào bộ nhớ đệm một sự từ chối khi nó có tính tất định cho khóa đã được xác thực đó; các nỗ lực không đúng định dạng, từng phần, tạm thời hoặc bị hủy bỏ cần chính sách thử lại rõ ràng.

Một bên chờ bị hủy không được làm mất hiệu lực của bên ngang hàng thành công. Bảo toàn ngữ cảnh lỗi của producer và lý do hủy của mỗi bên gọi. Giữ quyền sở hữu và chi phí dung lượng cho đến khi công việc dùng chung và dọn dẹp ổn định; bảo toàn các quan sát đã hoàn thành khi quá trình dọn dẹp thất bại. Thực thi việc vô hiệu hóa, các consumer đồng thời, hủy bên chờ cuối cùng, đóng và thử lại dọn dẹp khi các đường dẫn đó thay đổi.

Giới hạn nhóm tiến trình và trạng thái lưu giữ, không chỉ là heap cha hoặc số lượng worker. Tính đến sự mở rộng giải mã và tuần tự hóa trước khi cấp phát. Chọn giới hạn CPU, bộ nhớ, đồng thời và đĩa từ máy chủ và khối lượng công việc; dừng lại trước OOM và xác minh việc dọn dẹp thuộc sở hữu. Một tác vụ bị dừng bởi bộ bảo vệ là chưa hoàn thành. Không thay thế giới hạn bộ đệm truyền tải bằng một kho kiểm kê lưu giữ không giới hạn hoặc che giấu các kết quả từng phần đằng sau trạng thái thành công.

## So sánh các tác vụ đã hoàn thành và giới hạn nhận định (Compare completed tasks and limit the claim)

Chạy quy trình công khai thực tế bị ảnh hưởng trước và sau với các đầu vào, kết quả, độ bao phủ và ngữ nghĩa Evidence tương đương. Bao gồm các đầu vào nhỏ để chi phí thiết lập và sự suy giảm hiệu năng có thể nhìn thấy được. Phân biệt giữa phiên mới, bộ nhớ đệm runtime ấm và bộ nhớ đệm tệp hệ điều hành. Luân phiên hoặc lặp lại các lần chạy khi phương sai ảnh hưởng đến quyết định. Chọn các kiểm tra suy giảm ranh giới có ý nghĩa cho các trường hợp engine thực tế không thể tái hiện một cách đáng tin cậy; một engine giả mạo không đủ điều kiện cho một provider thực tế.

Báo cáo kết quả tác vụ, thời gian từng giai đoạn liên quan, tổng thời gian trôi qua và thời gian CPU, bộ nhớ đỉnh của nhóm tiến trình và phương pháp hạch toán của nó, các điểm dừng tài nguyên và quá trình dọn dẹp. Kiểm tra trạng thái thoát của payload cũng như của trình định hình (profiler). Giữ lại các lần thử thất bại và xác định các chi phí chưa được đo lường; một lần thử lại thành công không giải thích được lý do thất bại.

Đối với các nhận định về token, hãy đếm lược đồ/khám phá, đối số, câu trả lời, lần thử lại và các phần tiếp theo cần thiết để hoàn thành tác vụ. Gán nhãn các ước tính của tokenizer tách biệt khỏi mức sử dụng mô hình thực tế và ghi lại hành vi của client/danh mục. Chỉ riêng JSON nhỏ hơn không xác lập được việc sử dụng ít token mô hình hơn cũng như chất lượng điều tra tốt hơn. Không khẳng định lợi thế về phần mềm phức tạp mà không có các cuộc điều tra đại diện đã hoàn thành và tính đúng đắn/độ bao phủ có thể so sánh được.

Kết thúc với điểm nghẽn, thay đổi sở hữu, ngữ nghĩa được bảo toàn, các kiểm tra đã thực thi, đo lường trước/sau và các giới hạn còn lại. Giữ cho bất kỳ bản ghi tích hợp nào luôn cập nhật với phiên bản đã thử nghiệm và phân biệt công việc production đã merge với mã chỉ thử nghiệm. Dừng lại khi điểm nghẽn được chứng minh đã được giải quyết hoặc lợi ích tiếp theo vẫn chỉ là suy đoán.
