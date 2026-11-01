# Sổ cái nghĩa vụ tái cấu trúc (Reconstruction Obligation Ledgers)

`build_reconstruction_obligation_ledger` chuyển đổi các bản ghi Bằng chứng đã xác thực thành một danh sách tất định các xác nhận tái cấu trúc. Quy trình làm việc tương tự có sẵn dưới dạng:
- Công cụ MCP: `build_reconstruction_obligation_ledger`
- Lệnh CLI: `rea build-reconstruction-obligation-ledger`

Sổ cái mang tính thận trọng theo thiết kế. Các dữ kiện Đồ thị Ứng dụng tĩnh tạo ra các nghĩa vụ ứng viên; chúng không chứng minh hành vi runtime hoặc tiến trình. Một nghĩa vụ bắt buộc chỉ đóng lại khi một liên kết manifest cung cấp một chủ sở hữu duy nhất, bất kỳ kiểu parser/schema/domain bắt buộc nào, mọi fixture trường hợp bắt buộc và một bộ xác minh đạt chuẩn có thẩm quyền có thể so sánh với quan sát ban đầu. Bộ xác minh phải liệt kê ID nghĩa vụ và kết quả của nó phải hiện diện trong bundle Bằng chứng đầu vào. Mâu thuẫn, định nghĩa hoặc chủ sở hữu trùng lặp, các ẩn số tồn dư, thiếu phụ thuộc và thẩm quyền không khả dụng sẽ giữ cho việc đóng ở trạng thái mở hoặc thất bại.

Thẩm quyền tiến trình ban đầu yêu cầu Bằng chứng `controlled-replay` quan sát được với một provider thu thập tiến trình được hỗ trợ và tải trọng thu thập hợp lệ.

Mỗi bằng chứng fixture và bộ xác minh phải liên kết kết quả của nó với mã băm triển khai của chủ sở hữu chính xác và bản sửa đổi nghĩa vụ đang được đánh giá.

## Yêu cầu CLI hoàn chỉnh

Yêu cầu trống này hữu ích để kiểm tra tích hợp client:

```sh
rea build-reconstruction-obligation-ledger   '{
    "evidence_bundle": {
      "artifacts": [],
      "providers": [],
      "environments": [],
      "scenarios": [],
      "captures": [],
      "unknowns": [],
      "records": []
    },
    "reviewed_obligations": [],
    "manifest": {
      "bindings": [],
      "contradictions": []
    }
  }'   --json
```

Đối với một cuộc điều tra thực tế, hãy xuất một bundle Bằng chứng từ phiên làm việc và thay thế bundle trống. Các bản ghi Đồ thị Ứng dụng và thu thập tiến trình sẽ tự động tạo ra các ứng viên thận trọng. Bổ sung các nghĩa vụ đã xem xét khi một xác nhận bắt buộc không được đại diện bởi các nguồn đó.

Mỗi phần tử trong `reviewed_obligations` chứa `obligation_id`, `obligation_version`, `title`, `application_layer`, `family`, `target`, `required`, `required_case_kinds`, `required_original_authority`, `required_fixture_authority`, `required_verifier_authority`, `requires_parser_type`, `dependency_obligation_ids`, `residual_unknown_ids`, `unavailable_authority`, `required_next_evidence`, `disposition`, và `review_evidence_ids`.
