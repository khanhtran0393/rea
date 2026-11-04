---
name: rea-ci-tests-audit
description: Kiểm toán CI và các bộ kiểm thử của REA về tính đúng đắn, sự sai lệch fixture, kết quả vượt qua giả tạo (false passes) và hiệu năng đo lường được; sử dụng cho các cuộc điều tra bộ kiểm thử hoặc quy trình làm việc và các thay đổi về hiệu quả, thay vì kiểm thử tính năng thông thường hoặc một lỗi kiểm thử đơn lẻ.
---

# Kiểm toán CI và kiểm thử REA (REA CI and tests audit)

Xác định xem những bước kiểm tra nào vẫn phát hiện các lỗi suy giảm thực tế và những chi phí nào có thể được loại bỏ mà không làm mất sự bảo vệ đó. Kết quả có thể là một bản sửa lỗi đã được xác minh, một khuyến nghị có bằng chứng hỗ trợ hoặc một quyết định dừng rõ ràng.

Tuân theo `AGENTS.md`, [hướng dẫn kiểm thử](../../../docs/testing.md) và [quy tắc đóng góp](../../../CONTRIBUTING.md) của bản checkout. Sử dụng các lệnh hiện tại và chuỗi công cụ đã ghim của chúng, thay vì các lệnh được ghi nhớ từ một cuộc kiểm toán trước đó. Bảo toàn phạm vi được yêu cầu: chỉ điều tra đơn thuần không cấp quyền chỉnh sửa; kỹ năng này không cấp quyền phát hành issue, push hoặc merge. Ủy quyền hiện có của người dùng vẫn được áp dụng. Giữ các ghi chú kiểm toán và kết quả thô bên ngoài cây được theo dõi, thông thường trong thư mục `/tmp` dành riêng cho tác vụ.

## Thiết lập mốc cơ sở đã thực thi (Establish the executed baseline)

Tìm nạp khi người dùng yêu cầu nhánh main mới nhất. Ghim SHA mốc cơ sở và phiên bản quy trình làm việc được kiểm tra; sử dụng một worktree biệt lập khi thay đổi hoặc so sánh mã. Không sửa đổi bản checkout của tác vụ khác. Ghi lại các thay đổi bẩn (dirty changes) và lệnh chính xác, chuỗi công cụ, máy chủ, các bài kiểm thử đã chọn, instrument và các điều kiện tiên quyết. Một lần chạy cũ là bằng chứng cho phiên bản được thực thi của nó, ngay cả khi nhánh main tiến triển.

Lần vết các trigger quy trình làm việc, lựa chọn đường dẫn đã thay đổi, phụ thuộc công việc, các action có thể tái sử dụng, script của gói, khám phá bài kiểm thử, producer hiện vật và các cổng báo cáo cho đến các lệnh thực sự thực thi. Bao gồm các script, dữ liệu được tạo, đăng ký runtime và ranh giới tiến trình con bị thiếu trong đồ thị import. Kiểm tra các đường dẫn lựa chọn khẳng định và phủ định: các tệp bắt buộc phải được phát hiện một lần cho mỗi cấu hình kiểm thử dự định; các lần lặp lại cố ý về nền tảng/cấu hình là độ bao phủ riêng biệt. Một bài kiểm thử rõ ràng bị thiếu hoặc báo cáo bắt buộc bị thiếu không được phép chuyển sang màu xanh. Các luồng tùy chọn có thể bỏ qua; bảo toàn các điều kiện tiên quyết đã nêu của chúng.

Sử dụng hiện vật thời gian và lỗi hiện có để chọn các họ đáng ngờ. Chạy các kiểm tra liên quan để thiết lập mốc cơ sở; dành riêng một bộ kiểm thử cục bộ đầy đủ cho việc chẩn đoán CI diện rộng hoặc sự không chắc chắn mà các kiểm tra tập trung không thể giải quyết. Không biên dịch lại hoặc chạy lại toàn bộ bộ kiểm thử không thay đổi sau mỗi lần chỉnh sửa tài liệu.

## Chứng minh lỗi hoặc sự bảo vệ (Prove the failure or protection)

Đối với mỗi ứng viên, hãy nêu rõ hành vi quan sát được mong đợi và chủ sở hữu của nó, sau đó xây dựng trường hợp tái hiện nhỏ nhất đối với hành vi sản xuất. Đi theo producer và các consumer bị ảnh hưởng trước khi quyết định nơi sửa chữa.

- Đối với sự sai lệch (drift), hãy so sánh các fixture với các yêu cầu tiếp nhận hiện tại, phản hồi giao thức, hợp đồng có tên, phiên bản công cụ và các điểm vào công khai thực tế. Một mock trông có vẻ hợp lý có thể thất bại trước hành vi mà bài kiểm thử của nó tuyên bố bao phủ. Tìm kiếm toàn bộ họ fixture bị ảnh hưởng, bao gồm cả các chuỗi phiên bản khác. Bảo toàn các fixture cố ý bị sai định dạng và kiểm tra sự từ chối dự kiến của chúng.
- Đối với một bài kiểm thử màu xanh đáng ngờ, hãy can thiệp làm nhiễu hành vi liên quan trong một bản sao biệt lập hoặc tiêm vào một lỗi có kiểm soát. Xác minh rằng assertion hoặc cổng kiểm soát hiện có phát hiện ra nó. Một bài kiểm thử có thể chỉ lặp lại mock của nó, import mã bị bỏ rơi hoặc che giấu lỗi tiến trình con/trình kiểm tra. Khôi phục các biến đổi sau thử nghiệm.
- Đối với thiết lập dùng chung hoặc cắt tỉa, hãy ánh xạ các assertion riêng biệt tới các kịch bản lưu giữ của chúng. Bảo toàn các đầu vào bị sai định dạng, việc hủy bỏ, lỗi dọn dẹp, quyền hạn, dung lượng, danh tính và các tương tác đồng thời có ý nghĩa. Một lần chạy nhanh hơn bỏ qua một trường hợp hoặc làm suy yếu oracle của nó không phải là một sự cải tiến.
- Đối với độ bao phủ của provider, hãy tách biệt bằng chứng producer giả mạo, ranh giới và engine thực tế. Kiểm tra các trường hợp bỏ qua và điều kiện tiên quyết có tên; fixture không xác lập hành vi thực tế của Ghidra, LLDB, trình duyệt hoặc các provider khác.

Phân loại các phát hiện thành lỗi tái hiện được, chi phí được hỗ trợ, giả thuyết hoặc bằng chứng không khả dụng. Ghi lại lệnh cơ sở, đầu ra/trạng thái thoát quan sát được, kết quả mong đợi, phạm vi bị ảnh hưởng và giải thích cạnh tranh. Một lỗi trên nhánh main có thể đã tồn tại từ trước; điều đó tự nó không làm cho nó không liên quan đến cuộc kiểm toán CI. Gán lỗi PR bằng cách sử dụng phiên bản được thực thi và một mốc cơ sở sạch, không chỉ dựa trên sự giống nhau của chúng với một lỗi trước đó. Chỉ thử lại để kiểm tra một giải thích tạm thời cụ thể; giữ lại lỗi ban đầu và dừng lại khi việc lặp lại không thêm bằng chứng. Một lần thử lại thành công không xác lập được nguyên nhân gốc rễ hoặc bản sửa lỗi.

## Đánh giá hiệu quả mà không làm mất cổng kiểm soát (Assess efficiency without losing the gate)

Đọc [đo lường và kiểm tra CI](references/measurement-and-ci.md) khi điều tra sự chậm chạp, bộ nhớ đệm, thời gian chờ của runner, hiện vật dùng chung hoặc nhiễu log. So sánh công việc hoàn thành tương đương trước khi tuyên bố tăng tốc. Phân biệt thời gian thiết lập/build, thực thi kiểm thử, instrument độ bao phủ, chờ đợi phần phụ thuộc và lập lịch runner; nêu rõ những chi phí nào vẫn chưa được đo lường.

Ưu tiên loại bỏ sự chuẩn bị trùng lặp đã được chứng minh, tái sử dụng thiết lập bất biến và giảm công việc tiến trình con lặp đi lặp lại trước khi thu hẹp độ bao phủ tính đúng đắn. Bảo toàn các phiên độc lập nơi vòng đời hoặc tính đồng thời là oracle. Giữ cho các bài kiểm thử trực tiếp luôn tươi mới; một lần vượt qua trước đó được lưu trong bộ nhớ đệm không thể xác thực máy chủ hiện tại, nội dung provider đã cài đặt, quyền hạn hoặc trạng thái bên ngoài.

## Sửa chữa, xác minh và bàn giao trong phạm vi (Fix, verify, and hand off within scope)

Nếu việc triển khai được ủy quyền, hãy thực hiện thay đổi cấu trúc nhỏ nhất tại ranh giới sở hữu và thêm một bài kiểm tra hồi quy thất bại trên mốc cơ sở vì lý do chính đáng. Kiểm tra lại các consumer bị ảnh hưởng và các đường dẫn phủ định có ý nghĩa; xác minh việc vô hiệu hóa bộ nhớ đệm hoặc khám phá khi các cơ chế đó thay đổi. Đọc tài liệu Turbo đã cài đặt trước khi thay đổi cấu hình hoặc lệnh của nó.

Chạy các kiểm tra áp dụng của kho lưu trữ và kiểm tra kết quả CI hiện tại đầy đủ trước khi merge được ủy quyền. Bảo toàn độ bao phủ bắt buộc và các cổng khám phá. Không chặn một lỗi, thêm các lần thử lại toàn diện, tăng thời gian chờ hoặc xóa các assertion riêng biệt chỉ để làm cho CI có màu xanh. Các thay đổi đối với những kiểm soát này cần có lý do được chứng minh riêng và khả năng phát hiện lỗi được lưu giữ.
