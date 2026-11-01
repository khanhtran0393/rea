# Điều hòa tĩnh/runtime JavaScript (Static/Runtime Reconciliation)

REA có thể kết hợp Đồ thị Ứng dụng JavaScript (JavaScript Application Graph) tĩnh hiện có với một hoặc nhiều quan sát trình duyệt, Electron hoặc V8 Inspector của Node/Electron trong khi vẫn bảo toàn các thẩm quyền khác nhau của chúng. Công cụ MCP `reconcile_javascript_runtime` và lệnh CLI `rea reconcile-javascript-runtime` chấp nhận các bản ghi Bằng chứng và trả về Bằng chứng dẫn xuất chứa Đồ thị Ứng dụng JavaScript kết hợp cùng các phân loại khớp, mơ hồ, không khớp và unknown rõ ràng.

Thao tác không thực hiện I/O hệ thống tệp, trình duyệt hay Electron. Nó chỉ chấp nhận Bằng chứng đã được xác minh ngữ nghĩa được tạo bởi `analyze_javascript_application`, `inspect_web_page`, `inspect_electron_page`, `observe_javascript_runtime`, hoặc `capture_electron_scenario`.

## Quy trình làm việc

1. Chạy `analyze_javascript_application` đối với chính xác một lớp ứng dụng. Thư mục cache hoặc assets tùy chọn có thể được phân tích độc lập và cung cấp dưới dạng các lớp bổ sung.
2. Chạy `inspect_web_page`, `inspect_electron_page`, hoặc `observe_javascript_runtime` chỉ đính kèm, hoặc cung cấp Bằng chứng `capture_electron_scenario` hiện có.
3. Chuyển các bản ghi Bằng chứng kết quả cho `reconcile_javascript_runtime`. URL trình duyệt thường cần một ánh xạ tiền tố URL rõ ràng. Vị trí tệp Electron và V8 Inspector đã trích xuất được ánh xạ tự động khi các tệp runtime nằm dưới thư mục gốc đầu vào đã phân tích.

Đầu vào MCP có dạng sau:

```json
{
  "static_layers": [
    {
      "role": "application",
      "analysis": "ANALYZE_JAVASCRIPT_APPLICATION_EVIDENCE",
      "runtime_mappings": [
        {
          "kind": "url-prefix",
          "prefix": "https://example.test/assets/",
          "artifact_prefix": "dist"
        }
      ]
    }
  ],
  "runtime_observations": ["INSPECT_WEB_PAGE_EVIDENCE"]
}
```

`static_layers` yêu cầu chính xác một vai trò `application` và cho phép các vai trò `cache` và `assets` bổ sung. Ánh xạ `file-root` dịch một thư mục gốc hệ thống tệp runtime chuẩn sang một tiền tố tương đối của artifact. Ánh xạ `url-prefix` thực hiện tương tự cho một origin HTTP(S) và tiền tố đường dẫn chính xác.

## Quy tắc khớp (Matching Rules)

Các mục tiêu, khung, script và worker runtime trở thành các node đồ thị thuộc phạm vi thu thập. REA đánh giá các ứng viên tĩnh tương thích một cách thận trọng:
1. Một SHA-256 nguồn thu thập cộng với một vị trí được ánh xạ khớp sẽ tạo ra một kết quả khớp nội dung và vị trí.
2. Khi không có ứng viên tĩnh nào tồn tại tại một vị trí được ánh xạ, một mã băm nội dung chính xác duy nhất có thể khớp.
