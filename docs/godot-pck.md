# Kho lưu trữ Godot PCK

`inspect-artifact` và `extract-artifact` đọc các định dạng Godot PCK thông thường 1, 2, 3 và 4 mà không cần chạy trò chơi hoặc yêu cầu Godot. Godot 4.4.1, 4.6.1 và 4.7.1 chính thức lần lượt tạo ra các định dạng 2, 3 và 4 trong quá trình xác minh; các bản phát hành engine khác chưa được xác minh. Khả năng tương thích định dạng 1 tuân theo bố cục của [trình nạp Godot 3.5](https://github.com/godotengine/godot/blob/3.5/core/io/file_access_pack.cpp#L201-L242) và [trình xuất](https://github.com/godotengine/godot/blob/3.5/editor/editor_export.cpp#L1094-L1164) và được bao phủ bởi các fixture độc lập và nhúng tổng hợp; bản xuất Godot 3 thực tế chưa được thử nghiệm.

Định dạng 1 sử dụng tiêu đề 84 byte, không có cờ pack hoặc cờ thành viên, và độ lệch thành viên tuyệt đối trong toàn bộ tệp nguồn, bao gồm cả vật mang thực thi được nhúng. Các cờ và cơ sở tệp được báo cáo của nó bằng 0 vì các trường đó không có. Các bản ghi xóa có độ lệch bằng 0 vẫn không được hỗ trợ; REA không áp dụng các bản vá.

Quá trình kiểm tra giữ lại các dữ kiện tiêu đề và thư mục ban đầu, bao gồm phiên bản engine, cờ, số lượng tệp đã khai báo, đường dẫn nguồn, độ lệch và MD5 đã khai báo. Đối với một thành viên được khai báo là `res://scenes/main.gd`, `source_path` của lần xuất hiện giữ lại khai báo chính xác đó trong khi `logical_path` là `scenes/main.gd`, đường dẫn tương đối được sử dụng để tra cứu và trích xuất. Các byte văn bản thuần quan sát được nhận danh tính SHA-256 định địa chỉ theo nội dung. MD5 là một xác nhận tính toàn vẹn của container, tách biệt khỏi SHA-256 quan sát được. Sự không khớp sẽ thất bại theo mặc định; `--integrity-policy record-and-continue` giữ lại các byte quan sát được và một mâu thuẫn không đáng tin cậy rõ ràng. MD5 toàn số 0 là một xác nhận unknown, không phải mã băm đã được xác minh.

## CLI

Bắt đầu với một bản tóm tắt thu gọn và giữ lại bundle Bằng chứng hoàn chỉnh:

```sh
rea inspect-artifact /artifacts/game.pck --detail summary   --evidence_output /evidence/game.json --json
rea extract-artifact /artifacts/game.pck --json
```

Phân phối tóm tắt yêu cầu `--evidence_output` và không bao giờ ghi đè lên tệp đó. Phản hồi chứa Bằng chứng `summary` và biên nhận `evidence_bundle`. Không có `--detail summary`, quá trình kiểm tra giữ phân phối nội dòng hoàn chỉnh của nó. Trích xuất tạo một thư mục tạm thời mới và trả về đường dẫn của nó. Nó từ chối các đường dẫn không an toàn và xác minh danh tính container trước khi tạo đầu ra.

Cùng các lệnh CLI đó định vị một PCK ở đầu tệp hoặc thông qua trailer EOF 12 byte của nó trong một vật mang thực thi. Chúng không quét các tải trọng tùy ý hoặc phát hiện các pack đằng sau các chữ ký đuôi bổ sung. SHA-256 nội dung gốc bao gồm toàn bộ vật mang; phân loại nội dung của nó vẫn là native, trong khi bản kiểm kê ghi lại vai trò PCK và ranh giới pack chính xác. Dấu hiệu EOF chỉ là một ứng viên: kích thước không hợp lệ hoặc vị trí được trỏ tới không có tiêu đề `GDPC` sẽ giữ nguyên phân loại native. Khi trailer trỏ đến tiêu đề pack, nội dung PCK bị lỗi hoặc không hỗ trợ vẫn là một lỗi. Ranh giới nhận diện này tuân theo trình nạp Godot tại bản sửa đổi nguồn được ghim [06d1e7a](https://github.com/godotengine/godot/blob/06d1e7a34ccde4029eed53d018cfde77dfe771d5/core/io/file_access_pack.cpp#L2099-L2146). Khám phá `open_binary` native thông thường không thay đổi.

## MCP và các dữ kiện đã chọn

Đối với một pack độc lập, hãy gọi `open_binary` với đường dẫn tuyệt đối của nó, sau đó gọi `inspect_artifact` với `{"detail":"summary"}`. Quá trình kiểm tra hoàn chỉnh và bản kiểm kê lồng nhau của nó sẽ ở lại trong phiên làm việc. Sử dụng `normalized_result.parent_evidence_id` của bản tóm tắt làm nguồn cho các khung nhìn tiếp theo:

```json
{
  "name": "inspect_analysis_view",
  "arguments": {
    "source": {
      "kind": "retained-evidence",
      "evidence_id": "<parent_evidence_id>"
    },
    "view": {
      "kind": "inventory",
      "collection": "occurrences",
      "logical_paths": ["scripts/main.gd", "project.binary"],
      "fields": [
        "logical_path",
        "declared_integrity",
        "hash_status",
        "format_metadata"
      ],
      "offset": 0,
      "limit": 32
    }
  }
}
```

Các lần xuất hiện giữ lại `occurrence_id`, `artifact_id` và `parent_occurrence_id` ngay cả khi `fields` trống. Gom nhóm ID xuất hiện với `occurrence_ids`, hoặc nối các ID nội dung trả về với `collection: "nodes"`, `artifact_ids: [...]` và `fields: ["sha256", "size"]`. Các bộ lọc giao nhau; bộ lọc bị bỏ qua sẽ chọn tất cả các hàng có sẵn và một mảng trống sẽ không chọn hàng nào. Phân trang tuân theo thứ tự chuẩn đã lọc. Nhiều lần xuất hiện có thể chia sẻ một ID nội dung.

Các khung nhìn tái sử dụng các dữ kiện kiểm kê bất biến đã xác thực và các chỉ mục tra cứu lười; chúng không mở lại nguồn. Đây là các quan sát lịch sử, bao gồm cả sau khi nguồn thay đổi hoặc biến mất. Đóng mục tiêu bằng `close_binary` khi hoàn tất. Để đọc một bundle đã xuất thông qua CLI, hãy chọn bản ghi kiểm tra hoàn chỉnh của nó làm `source: {kind: "inline", evidence: ...}` trong tệp yêu cầu JSON cho `rea inspect-analysis-view request.json --json`.
