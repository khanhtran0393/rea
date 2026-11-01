# Giải mã tài nguyên bằng Apktool (Apktool Resource Decoding)

REA giải mã các bảng tài nguyên Android và manifest nhị phân thông qua trình khởi chạy Apktool của bên gọi. Điều này hoàn thiện bộ ba tĩnh cho các mục tiêu Android: mã nguồn Java/Kotlin thông qua JADX, dịch ngược cấp engine thông qua JEB, và tài nguyên thông qua Apktool. REA không bao giờ cài đặt Apktool hay Java runtime.

## Cung cấp công cụ

Sử dụng Apktool 2.7.0 trở lên (đã xác minh với 2.7.0 và 3.0.3 trên Linux) và Java runtime mà trình khởi chạy có thể tìm thấy:

```sh
# Tùy chọn: đường dẫn tuyệt đối đến trình khởi chạy apktool
export REA_APKTOOL_COMMAND=/absolute/path/apktool
```

Kiểm tra tính sẵn sàng bằng `rea inspect-apktool-client`: lệnh chạy `apktool --version` và báo cáo trình khởi chạy đã được giải quyết, nguồn lựa chọn và phiên bản của nó.

Trên Windows, trình khởi chạy `.bat` hoặc `.cmd` chạy thông qua lệnh gọi `cmd.exe /d /s /c` rõ ràng, không có AutoRun, không có delayed expansion và trích dẫn mọi đối số.

## Trả lời câu hỏi phân tích

```sh
rea inspect-apktool-client
rea decode-android-resources /targets/Example.apk
rea decode-android-resources /targets/Example.apk --locale de
rea decode-android-resources /targets/Example.apk --locales de --locales pt-BR
rea decode-android-resources /targets/Example.apk --locales de --locales pt-BR --string-tables-only
rea decode-android-resources /targets/Example.apk --no-strings
rea decode-android-resources /targets/Example.apk --decode-timeout-ms 900000
```

Các phương thức MCP tương đương là `inspect_apktool_client` và `decode_android_resources`. Yêu cầu giải mã nhận `path`, `locale` chính tùy chọn (chiếu `res/values-<locale>/strings.xml` thay vì bảng mặc định), `locales` (mảng các bảng chuỗi đã bản địa hóa bổ sung để chiếu từ cùng một lượt giải mã đơn lẻ), và `include_strings` (mặc định true).

Đặt `string_tables_only: true` (CLI `--string-tables-only`) để chỉ trả về các mục chuỗi bên trong `string_tables`. Mảng `strings` cấp cao nhất khi đó sẽ trống theo yêu cầu.

## Diễn giải kết quả

- **Một snapshot APK thuộc sở hữu được băm và giải mã.** `target.sha256` và `target.bytes` liên kết quan sát với APK chính xác đã được giải mã.
- **Metadata đến từ apktool.yml như apktool đã in.** Mã phiên bản, tên gói và cờ nén được bảo toàn.
