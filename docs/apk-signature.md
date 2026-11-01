# Các sơ đồ chữ ký APK (APK Signature Schemes)

REA đọc chữ ký của APK Android (hoặc ZIP/AAB) cục bộ được chọn một cách tĩnh để nhà phân tích có thể trả lời **ai đã ký tệp này, theo những sơ đồ nào, và liệu mã băm nội dung đã khai báo có khớp với các byte này hay không** trước khi tin cậy hoặc phân loại nó. Bộ đọc chạy trong tiến trình và không cần engine bên ngoài: nó phân tích cú pháp phần đuôi ZIP và Khối Chữ ký APK (APK Signing Block), đọc chứng chỉ bằng bộ phân tích X.509 nền tảng và tính toán lại mã băm nội dung. Không có mã Android nào được thực thi, và không liên quan đến thiết bị hay `adb`.

CLI và MCP chia sẻ cùng một quy trình làm việc và trả về Bằng chứng gắn với artifact nội dòng.

```sh
rea inspect-apk-signature /path/app.apk
```

## Dữ kiện đã chọn và Bằng chứng được giữ lại

MCP mặc định trả về bản tóm tắt với sự hiện diện của sơ đồ, dấu vân tay chứng chỉ của người ký, số lượng hàng, trạng thái và độ bao phủ rõ ràng. `detail: "complete"` trả về báo cáo đầy đủ:

```json
{
  "path": "/path/app.apk",
  "view": {
    "collection": "signers",
    "schemes": ["v3"],
    "fields": ["certificate_sha256"],
    "limit": 32
  }
}
```

Các collection bao gồm `signers`, `digests` và các node `lineage` đã khai báo.

## Những gì công cụ báo cáo

- **Các sơ đồ hiện diện:** v1 (ký JAR, từ thư mục trung tâm) và v2, v3, v3.1 của APK Signing Block.
- **Mỗi người ký v1:** REA giải nén khối và đọc `SignedData` PKCS#7 của nó.
- **Mỗi người ký v2/v3:** Đọc các cặp ID-value trong APK Signing Block, trích xuất chứng chỉ X.509 và các thuật toán băm/chữ ký.
