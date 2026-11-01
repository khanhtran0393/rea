# Nâng cấp từ REA 6.1 lên 6.2

REA 6.2 bao gồm các bản sửa lỗi về độ ổn định và các thay đổi đối với kết quả công khai và hợp đồng dữ liệu đã lưu. Kho lưu trữ sử dụng `always-bump-minor`, bao gồm cả các bản phát hành có thay đổi phá vỡ (breaking); việc tăng phiên bản phụ không ngụ ý tính tương thích ngược. Làm mới các định nghĩa công cụ MCP được lưu trong bộ nhớ cache sau khi nâng cấp và điều chỉnh các consumer tùy chỉnh theo hướng dẫn này.

## Kết quả MCP và Bằng chứng (Evidence)

Các công cụ tạo Bằng chứng hiện trả về bản ghi Bằng chứng hoàn chỉnh trực tiếp trong `structuredContent` và văn bản JSON. Lớp bao bọc `{ result, evidence_id, evidence }` cũ đã bị xóa.
Đối với lệnh gọi tạo Bằng chứng thành công, hãy thay đổi:

```javascript
// 6.1
const value = reply.structuredContent.result;
const evidence = reply.structuredContent.evidence;

// 6.2
const evidence = reply.structuredContent;
const value = evidence.normalized_result;
const evidenceId = evidence.evidence_id;
```

Kiểm tra `isError` trước khi đọc các trường thành công. Kết quả lỗi có `isError: true` và mang chẩn đoán JSON có kiểu của chúng trong một khối nội dung văn bản; chúng bỏ qua `structuredContent` để không vi phạm schema đầu ra thành công đã quảng bá.

```javascript
if (reply.isError === true) {
  const text = reply.content.find((part) => part.type === "text");
  if (text === undefined) throw new Error("Lỗi REA không có chẩn đoán văn bản");
  const { error: diagnostic } = JSON.parse(text.text);
  // Xử lý diagnostic.code, diagnostic.message và diagnostic.details.
}
```

Các công cụ vòng đời và công cụ không tạo Bằng chứng khác vẫn sử dụng các hợp đồng đầu ra đã quảng bá của riêng chúng. Không cho rằng mọi công cụ thành công đều trả về Bằng chứng. Tham khảo [`tools/list` và hướng dẫn kết quả](mcp-contracts.md#tool-results).

Lỗi vận chuyển `resource_constraint` có thể bao gồm một `details.reported_limits.evidence_reference` được giữ lại. Sử dụng `inspect_analysis_view` hoặc `export_evidence_bundle` trên cùng một kết nối để tiêu thụ Bằng chứng được giữ lại hoàn chỉnh.

Mỗi bản ghi Bằng chứng hiện yêu cầu `analysis_profile`, là đối tượng profile quan sát được của nó hoặc `null`. Trường này tham gia vào danh tính Bằng chứng.

Kết quả `binary_session` loại bỏ các bí danh cấp cao nhất `provider` và `providers` dư thừa. Đọc `analysis_provider_binding`, `analysis_provider_candidates` và `capabilities` theo từng thao tác để chọn và xác định tính khả dụng.

## Snapshot đã lưu và kiểm kê artifact

Snapshot phân tích yêu cầu `workflow_entries` và một bản ghi Bằng chứng ràng buộc profile tương ứng cho mọi truy vấn được lưu vào cache. Import từ chối các ràng buộc chưa đầy đủ thay vì âm thầm loại bỏ chúng.

Node kiểm kê artifact xác định các byte nội dung. Vai trò và quyền theo đường dẫn cụ thể hiện thuộc về từng lần xuất hiện:

| Đã đọc trước đây                | Đọc trong 6.2                             |
| -------------------------------- | ----------------------------------------- |
| `nodes[].kind`                   | `occurrences[].artifact_kind`             |
| `nodes[].format` theo đường dẫn  | `occurrences[].artifact_format`           |
| `nodes[].executable`             | `occurrences[].executable`                |
| Danh tính nội dung và định dạng byte | `nodes[].artifact_id`, `sha256`, `format` |

Nối một lần xuất hiện với node nội dung của nó bằng `artifact_id`; kiểm tra vai trò của lần xuất hiện đã chọn thay vì chọn một vai trò cho tất cả các đường dẫn có byte bằng nhau.
