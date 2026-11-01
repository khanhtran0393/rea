# Khai báo tệp mô hình AI (Model File Declarations)

Sử dụng `inspect_model_artifact` (CLI: `inspect-model-artifact`) để kiểm tra tệp GGUF v2/v3 hoặc safetensors cục bộ. Việc phát hiện sử dụng nội dung, bao gồm thứ tự byte của GGUF, thay vì phần mở rộng của nó. REA không tải mô hình, không diễn giải trọng số, không chạy suy luận, không import Python, không mở tham chiếu dữ liệu bên ngoài và không sử dụng mạng.

Các phiên bản GGUF không được hỗ trợ sẽ bị từ chối. Little-endian v2/v3 và big-endian v3 được hỗ trợ; big-endian v2 bị từ chối.

```bash
rea inspect-model-artifact /absolute/path/model.gguf --json
rea inspect-model-artifact /absolute/path/model.safetensors --hash full --json
```

Đối với CLI, mặc định là Bằng chứng hoàn chỉnh. Đối với phân phối thu gọn:

```bash
rea inspect-model-artifact /absolute/path/model.gguf   --detail summary --evidence_output /absolute/path/model-evidence.json --json
```

## Danh tính và việc đọc

Mặc định `hash: "header"` chỉ đọc metadata và thư mục tensor. Kết quả ghi lại đường dẫn chính xác, kích thước tệp, độ dài tiêu đề, SHA-256 tiêu đề, số byte đã đọc và phạm vi đọc.

`hash: "full"` bổ sung việc truyền các byte tệp còn lại qua SHA-256. Nó trả về mã băm toàn bộ tệp và chủ thể Bằng chứng trong khi vẫn giữ nguyên mã băm tiêu đề đó.

## Dữ kiện và truy vấn đã chọn

GGUF báo cáo phiên bản, thứ tự byte, căn chỉnh, metadata có kiểu và tên của từng tensor, ID loại ban đầu, kích thước, độ lệch dữ liệu tương đối, độ lệch tệp và độ dài byte dẫn xuất.
