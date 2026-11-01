# Chiếu kiểm kê ứng dụng Apple (Apple Application Inventory Projection)

REA có thể chiếu một hoặc nhiều bản ghi Bằng chứng `inventory_artifact` đã xác thực cho cùng một thư mục IPA, `.app` của macOS, ZIP hoặc DMG vào một bản kiểm kê ứng dụng Apple. Phép chiếu hoàn toàn không thực thi: nó báo cáo các đường dẫn chính xác, mã băm nội dung, định dạng được phát hiện, cấu trúc bundle, gợi ý họ runtime và giả thuyết cầu nối dựa trên đường dẫn.

```sh
rea project-apple-application-graph '{"inventory_evidence":[<inventory_artifact Evidence>]}'
```

```json
{
  "name": "project_apple_application_graph",
  "arguments": {
    "inventory_evidence": ["<inventory_artifact Evidence>"]
  }
}
```

Kết quả bao gồm mọi thành phần trong từng danh mục kiểm kê. Các giả thuyết cầu nối là các cặp JavaScript/native dựa trên đường dẫn trong mỗi thư mục gốc ứng dụng.

## Thư mục gốc ứng dụng (Application Roots)

| Gốc kiểm kê                           | Gốc ứng dụng                                        |
| ------------------------------------- | --------------------------------------------------- |
| IPA                                   | Mỗi `Payload/<Name>.app`                            |
| Thư mục `.app`                        | `.`, chính bundle đã được kiểm kê                   |
| Thư mục, ZIP hoặc DMG chứa các ứng dụng | Mỗi `<Name>.app` ngoài cùng theo sau là `Contents/` |

Các tệp được kiểm kê nhưng byte của chúng không được đọc, chẳng hạn như các thành viên lưu trữ được mã hóa, được liệt kê trong `unavailable_components` với đường dẫn ban đầu, định dạng, cờ mã hóa, giới hạn và danh tính nội dung null.
