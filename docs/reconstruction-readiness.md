# Xác minh tính sẵn sàng tái cấu trúc (Reconstruction Readiness)

Chạy các luồng phân tích tĩnh công khai và kiểm tra bộ đánh giá báo cáo tổng hợp từ một bản checkout sạch:

```sh
npm ci
npm run verify:readiness
```

Lệnh này xây dựng các fixture tuân thủ native có thể phân phối lại, kiểm tra tệp thực thi native thông qua CLI công khai, và phân tích các nguồn fixture JavaScript CLI và Electron thông qua cả CLI và MCP. Nó kiểm tra tính khớp của các danh tính Bằng chứng CLI/MCP cho hai phân tích ứng dụng đó. Các ứng dụng fixture không được thực thi và tính tương đương MCP native không được thử nghiệm.

Tách biệt khỏi điều đó, lệnh đánh giá một đầu vào 9 giai đoạn hoàn toàn tổng hợp và kiểm tra tính tất định của bộ đánh giá, việc từ chối Bằng chứng bị giả mạo và việc từ chối mã băm nguồn cũ. Các kiểm tra này liên quan đến việc đánh giá báo cáo; chúng không xác lập hành vi runtime của ứng dụng, phục hồi thẩm quyền, dọn dẹp hủy bỏ, tính tương đương hành vi hoặc đóng tái cấu trúc cho các fixture được phân tích. Lệnh in một bản tóm tắt JSON nhỏ gọn.

Đặt `REA_READINESS_REPORT_PATH` để giữ lại báo cáo máy đọc được hoàn chỉnh:

```sh
REA_READINESS_REPORT_PATH=readiness-report.json npm run verify:readiness
```

CI chạy lệnh này trên Linux, xuất bản bản tóm tắt thu gọn và giữ lại báo cáo đầy đủ dưới dạng artifact `reconstruction-readiness`.

## Phạm vi báo cáo và quyền sở hữu

Bộ xác minh chạy dưới dạng `npm run verify:readiness`; nó không được hiển thị dưới dạng lệnh CLI công khai hoặc công cụ MCP. Báo cáo của nó tách biệt hai loại xác minh:

- `observed_journeys` giữ lại Bằng chứng phân tích tĩnh thực tế và các kiểm tra danh tính CLI/MCP quan sát được, cùng giới hạn độ bao phủ của chúng.
- `evaluator_conformance` giữ lại báo cáo đường cơ sở tổng hợp và các kiểm tra của bộ đánh giá. `pass` 9 giai đoạn của nó mô tả việc chấp nhận các quan sát tổng hợp được cung cấp.

Artifact được giữ lại hiện sử dụng `schema: "ReconstructionReadinessVerificationReport"` và `schema_version: 1`. Báo cáo 9 giai đoạn tổng hợp hiện nằm tại `evaluator_conformance.baseline_report`. Bằng chứng CLI và MCP quan sát được được giữ lại trong các bundle riêng biệt dưới `observed_journeys.evidence_bundles`.

## Các giai đoạn của bộ đánh giá tổng hợp

| Giai đoạn                       | Các quan sát bắt buộc được cung cấp                                                                                   |
| ------------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| `discover-classify`             | khởi động không mục tiêu, kiểm kê artifact, chọn luồng tương thích và giới hạn hiển thị chính xác                      |
| `diagnose-environment`          | tách biệt runtime hỏng khỏi lỗi phân tích, phát hiện lệch phiên bản và thay đổi bản sửa đổi sau phục hồi              |
| `acquire-authority`             | báo cáo phạm vi thiếu, tiếp tục cấp quyền hạn hẹp, nhắc lại việc mở rộng và từ chối không khởi chạy gì                |
| `static-analysis`               | định tuyến native và JavaScript cùng bảo toàn tính mơ hồ, cắt ngắn, cú pháp không hỗ trợ và ẩn số tồn dư            |
| `reactive-scenarios`            | tương tác dựa trên vị ngữ, lắng đọng con cái, tương quan Electron và Bằng chứng dọn dẹp hủy bỏ                        |
| `compare-authority-candidate`   | ngữ nghĩa dấu vết hữu hạn/thứ tự một phần rõ ràng, định vị phân kỳ chính và unknown cho so sánh chưa hoàn tất        |
| `preserve-contradictions`       | ghi nhận và tiếp tục, chặn tương đương bị ảnh hưởng và giữ lại so sánh không bị ảnh hưởng                             |
| `verify-reconstruction-closure` | phát hiện tái cấu trúc chưa hoàn tất, đóng xác minh tăng dần và từ chối các ứng viên ủy quyền thẩm quyền             |
| `export-replay`                 | tính tương đương CLI/MCP, lỗi cấu trúc/tiến trình, phát lại tất định, từ chối giả mạo và từ chối đầu vào cũ            |

## Quy tắc Fail-Closed

Bộ đánh giá từ chối hoặc giữ lại báo cáo tổng hợp đạt chuẩn khi:
- một năng lực có giới hạn bỏ qua các giới hạn số hiệu dụng của nó;
- một fixture thiếu chính xác một luồng tương thích được chọn;
- một provider không tương thích hoặc không khả dụng được gọi;
- một lỗi CLI có kiểu thoát với mã 0, hoặc một lỗi MCP báo cáo thành công;
- hành vi đồng thời sử dụng so sánh toàn phần;
- việc cắt ngắn, mất ổn định, thẩm quyền không khả dụng hoặc mâu thuẫn được dán nhãn là tương đương;
- phát lại không tất định, giả mạo/cũ không được phát hiện, hoặc mã băm nguồn dự kiến khác biệt.
