# Kiểm tra và thu thập thiết bị qua ADB

REA kiểm tra các thiết bị Android đã kết nối, đọc trạng thái thiết bị và truyền tệp cũng như gói thông qua tệp nhị phân `adb` của bên gọi. Trình giả lập, thiết bị USB và thiết bị mạng đều tham gia như nhau: REA không tự khởi động hay quản lý chúng và không bao giờ cài đặt các công cụ nền tảng.

## Cung cấp công cụ

Sử dụng bất kỳ `adb` nào của Android platform-tools (đã xác minh với 34.0.5). Chọn nó thông qua biến môi trường:

```sh
export REA_ADB_PATH=/absolute/path/to/adb
```

Tính sẵn sàng được kiểm tra bằng `rea inspect-adb-client`: lệnh chạy `adb version` và báo cáo tệp nhị phân đã được giải quyết, phiên bản của nó và nơi nó được chọn.

## Trả lời câu hỏi phân tích

```sh
# Định danh thiết bị
rea inspect-adb-client
rea list-adb-devices
rea inspect-adb-device emulator-5554
rea inspect-adb-display emulator-5554
rea list-adb-features emulator-5554
rea list-adb-services emulator-5554

# Tìm và thu thập ứng dụng
rea resolve-adb-packages emulator-5554 whatsapp
rea list-adb-packages emulator-5554 --scope third_party
rea inspect-adb-package emulator-5554 com.whatsapp
rea pull-adb-package emulator-5554 com.whatsapp --output-directory /tmp/acq
rea inspect-android-package /tmp/acq/com.whatsapp/base.apk

# Trạng thái thiết bị và tệp
rea list-adb-processes emulator-5554
rea inspect-adb-window emulator-5554
rea read-adb-setting emulator-5554 global adb_enabled
rea read-adb-logcat emulator-5554 --count 200
rea list-adb-directory emulator-5554 /data/data/com.whatsapp
rea pull-adb-file emulator-5554 /data/local/tmp/notes.txt --output-directory /tmp/acq
rea push-adb-file emulator-5554 /tools/frida-server /data/local/tmp/frida-server
rea capture-adb-screen emulator-5554 --output-directory /tmp/shots
rea collect-adb-bugreport emulator-5554 --output-directory /tmp/bugs

# Vòng đời gói và phân tích động (thay đổi thiết bị)
rea install-adb-package emulator-5554 /tmp/app.apk --replace
rea start-adb-app emulator-5554 com.example.app
rea read-adb-logcat emulator-5554 --pid 12345
rea start-adb-activity emulator-5554 android.intent.action.VIEW --data-uri https://example.com
rea stop-adb-app emulator-5554 com.example.app
rea uninstall-adb-package emulator-5554 com.example.app
```

Các phương thức MCP tương đương sử dụng tên snake_case (`list_adb_devices`, `pull_adb_package`, v.v.).
