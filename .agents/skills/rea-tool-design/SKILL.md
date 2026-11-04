---
name: rea-tool-design
description: Kiểm toán, điều tra, thiết kế hoặc thay đổi các công cụ điều tra của REA, hợp đồng CLI/MCP, khả năng của provider và ngữ nghĩa Evidence.
---

# Thiết kế công cụ REA (REA tool design)

Sử dụng [hướng dẫn thiết kế công cụ](../../../docs/tool-design.md) để xác định ranh giới công cụ, ngữ nghĩa hợp đồng, quyền sở hữu provider và khả năng khám phá (discoverability). Kiểm tra hợp đồng hiện tại và công cụ hiện có gần nhất liên quan đến câu hỏi của nhà phân tích; mã nguồn, tài liệu và danh mục trực tiếp có thể mô tả các phiên bản khác nhau.

Đối với công việc phụ thuộc vào provider, hãy thực hiện tác vụ của nhà phân tích thông qua REA và so sánh các quan sát từ provider native tương ứng cùng các hiện vật độc lập nếu có. Kiểm tra các kết quả từng phần có thể áp dụng, ranh giới phục hồi và phiên làm việc; báo cáo những gì chưa được thực thi. Sử dụng [xác minh quy trình làm việc](../../../docs/testing.md#provider-backed-workflow-verification) để chọn bằng chứng cho hợp đồng đề xuất thay vì tìm kiếm sự tương đồng tuyệt đối về API của provider.

Đối với một cuộc kiểm toán (audit), hãy làm theo [kiểm toán các công cụ hiện có](../../../docs/tool-design.md#audit-existing-tools): xác định phạm vi các tác vụ đại diện, thực thi quy trình trực tiếp, phân biệt giữa lỗi tái hiện được, các giới hạn được hỗ trợ, cơ hội thiết kế và các nhận định chưa được xác minh. Báo cáo các giải pháp thay thế khả dụng và giới hạn bằng chứng cũng như các phát hiện.

Đối với yêu cầu thiết kế, hãy trả về ranh giới đề xuất, cấu trúc đầu vào/kết quả, ngữ nghĩa Evidence, giải pháp thay thế gần nhất và phương pháp xác minh. Triển khai khi được yêu cầu, bảo toàn phạm vi của người dùng và các ủy quyền hiện có.

Đối với việc triển khai hoặc xác minh, hãy tham khảo [kiểm thử](../../../docs/testing.md) cho các luồng consumer và provider bị ảnh hưởng, và [đóng góp](../../../CONTRIBUTING.md) cho quyền sở hữu tệp được tạo tự động. Chỉ đọc hướng dẫn provider/quy trình khi tác vụ yêu cầu.
