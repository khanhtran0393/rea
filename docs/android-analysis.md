# Phân tích Tĩnh Tệp APK Android (Static Android APK Analysis)

REA tích hợp công cụ JADX không giao diện (headless JADX engine) thông qua `src/android/`. Mã nguồn của nó được duy trì dưới dạng Git submodule cố định commit; xem [nguồn gốc upstream](https://github.com/morluto/rea/blob/main/third_party/README.md). CLI và MCP sử dụng cùng một quy trình ứng dụng và trả về Bằng chứng gắn liền với artifact.

Họ công cụ này có mặt trên nhánh `main` và trong bản phát hành npm 4.1.0. Kiểm tra [ranh giới gói phát hành](installation.md#released-package-and-main) và các phụ thuộc liên quan.

```sh
rea inspect-apk ./app.apk --json
rea inspect-apk ./app.apk --decompile-sources --output-dir ./decompiled/
```

## Phân phối gói tùy chọn

Bạn có thể phân tích các biến thể APK hoặc gói phân phối:
- Tệp `.apk` đơn lẻ tiêu chuẩn.
- Gói ứng dụng chia nhỏ (Split APKs / `.apks`).
- Bản sao lưu ứng dụng hoặc dữ liệu trích xuất từ thiết bị.

## Phép chiếu thành phần Manifest (Manifest Component Projection)

REA tự động phân tích cú pháp `AndroidManifest.xml` đã được giải mã nhị phân:
- **Activities**: Các màn hình giao diện, cờ `exported`, bộ lọc ý định (`intent-filters`), quyền bảo vệ.
- **Services**: Các dịch vụ chạy ngầm, bộ lắng nghe thông báo, bộ đồng bộ hóa.
- **Broadcast Receivers**: Các bộ thu sóng sự kiện hệ thống và nội bộ.
- **Content Providers**: Các nhà cung cấp nội dung dữ liệu, quyền URI, khả năng cấp quyền đọc/ghi.
- **Quyền hạn (Permissions)**: Các quyền thông thường và nguy hiểm mà ứng dụng yêu cầu (`uses-permission`).

## Cung cấp công cụ và Môi trường Java

JADX yêu cầu môi trường Java Runtime (JRE/JDK 11+). REA tự động tìm kiếm Java trên `PATH`. Bạn có thể chỉ định JDK tùy chỉnh:

```sh
# Tùy chọn: chọn JDK hiện có thay vì java trên PATH
export JAVA_HOME=/path/to/custom/jdk
rea inspect-apk ./app.apk --json
```

## Trả lời câu hỏi của chuyên gia phân tích (Answer an Analyst Question)

Thông qua giao thức MCP, tác tử có thể truy vấn ngữ nghĩa cấp cao:
- "Ứng dụng này giao tiếp với các máy chủ nào?"
- "Lớp nào xử lý việc xác thực hoặc mã hóa mã khóa?"
- "Có thành phần nào được `exported` mà không có quyền bảo vệ không?"

## Áp dụng ánh xạ R8/ProGuard (Apply an R8/ProGuard Mapping)

Nếu bạn có tệp ánh xạ giải mã làm mờ (mapping file):

```sh
rea inspect-apk ./app.apk --mapping ./mapping.txt --json
```

REA sẽ khôi phục tên lớp, phương thức và trường gốc trong toàn bộ cây phân tích mã nguồn.

## Liên kết khai báo Native với các ứng viên JNI

Đối với các ứng dụng sử dụng mã máy C/C++ (`.so` trong thư mục `lib/`):
- REA tự động xác định các phương thức `native` trong Java.
- Ánh xạ chữ ký phương thức tới các biểu tượng JNI trong thư viện nhị phân: `Java_com_example_app_NativeLib_methodName`.
- Phát hiện các hàm đăng ký động `RegisterNatives` trong `JNI_OnLoad`.

## Diễn giải kết quả phân tích

Đầu ra Evidence bao gồm:
- **Độ tin cậy (Confidence)**: Mức độ chắc chắn của quá trình dịch ngược và phân tích luồng điều khiển.
- **Danh sách nhị phân phụ thuộc**: Các tệp `.so` được phát hiện và kiến trúc ABI (`arm64-v8a`, `armeabi-v7a`, `x86_64`).
- **Điểm nóng rủi ro (Risk hotspots)**: Sử dụng mã hóa không an toàn, HTTP văn bản thuần, lưu trữ ngoài không bảo vệ.

## Giới hạn tài nguyên và vòng đời

- REA sử dụng quy trình con độc lập cho JADX để bảo vệ tiến trình chính không bị cạn kiệt bộ nhớ JVM.
- Giới hạn bộ nhớ heap mặc định của JADX là 4GB (có thể định cấu hình qua cờ `--jvm-max-memory`).
- Thời gian chờ phân tích tĩnh mặc định là 300 giây cho các tệp APK lớn (>100MB).

## Xác minh thực tế (Real Verification)

REA duy trì bộ kiểm thử xác minh đối với các ứng dụng mẫu mã nguồn mở phổ biến để đảm bảo tính ổn định và không suy giảm hiệu năng giữa các phiên bản.
