# Nhận diện bản dựng Flutter (Flutter Build Identification)

REA nhận diện các dữ kiện bản dựng của ứng dụng Flutter bằng cách **phân tích cú pháp thuần túy** APK của nó: mã băm snapshot Dart VM, mã băm thư viện theo từng ABI, GNU build-id của engine và bất kỳ chuỗi phiên bản nào được gắn nhãn. Không có engine, không cài đặt công cụ, không thực thi và không tra cứu bên ngoài — họ công cụ này trả lời "đây là bản dựng Dart SDK nào, và chính xác có gì trong tải trọng?" trước bất kỳ phân tích native sâu hơn nào.

Đây là bước phân loại cho việc dịch ngược Flutter: mã băm snapshot là artifact ánh xạ tới một bản phát hành Dart SDK thông qua tập dữ liệu bên ngoài (chẳng hạn như tập dữ liệu mà reFlutter duy trì), và việc biết phiên bản SDK là điều làm cho các trình phân tích snapshot đặc thù theo bố cục SDK có thể áp dụng được.

## Trả lời câu hỏi phân tích

```sh
rea identify-flutter-build /targets/Example.apk
rea inspect-dart-aot /targets/Example.apk
rea inspect-dart-aot /targets/Example.apk --abi armeabi-v7a
```

Các phương thức MCP tương đương là `identify_flutter_build` (với `path`) và `inspect_dart_aot` (với `path` và `abi` tùy chọn; mặc định ưu tiên arm64-v8a, sau đó là x86_64, x86, armeabi-v7a, rồi đến các ABI còn lại theo thứ tự bảng chữ cái).

## Phân phối AOT được giữ lại tùy chọn

MCP `inspect_dart_aot` giữ đầu ra hoàn chỉnh khi bỏ qua `detail` hoặc đặt `"complete"`. Đặt rõ ràng `detail: "summary"` trả về danh tính mục tiêu/ABI/thư viện đã chọn, tất cả tiêu đề snapshot, độ bao phủ cấu trúc, độ bao phủ string-pool và số lượng giới hạn dưới ban đầu mà không có danh sách chuỗi.

Sử dụng `normalized_result.parent_evidence_id` làm nguồn được giữ lại cho `inspect_analysis_view`, với `view: {kind: "dart-aot", collection: "package_uris", offset: 0, limit: 64}`. Các collection là `package_uris`, `dart_uris` và `class_like_tokens`.

## Những gì quá trình kiểm tra AOT trả về

`inspect_dart_aot` đọc toàn bộ `libapp.so` của một ABI và báo cáo:
- **Các section snapshot** — 4 biểu tượng `_kDart*Snapshot*` được định vị thông qua **bảng biểu tượng động ELF** (ELF32 và ELF64, little-endian), mỗi biểu tượng có độ lệch tệp, kích thước và — đối với các section dữ liệu mang magic `f5f5dcdc` — loại và độ dài tiêu đề đã phân tích. Độ bao phủ cấu trúc hoàn chỉnh hiện yêu cầu kind 3, giá trị FullAOT trong [bố cục Dart 2.14](https://github.com/dart-lang/sdk/blob/2.14.0/runtime/vm/snapshot.h) được hỗ trợ.
- **Mã băm snapshot** từ tiêu đề ở đầu mỗi biểu tượng dữ liệu được xuất. Độ bao phủ mã băm hoàn chỉnh yêu cầu tiêu đề hợp lệ từ cả VM và isolate.
- **Một phép chiếu string-pool từ section dữ liệu isolate**: các URI nguồn `package:` riêng biệt (các tệp Dart của chính ứng dụng và các phụ thuộc của nó — một bản kiểm kê phụ thuộc), các URI SDK `dart:` (các thư viện SDK mà ứng dụng sử dụng) và các token dạng định danh, mỗi loại có giới hạn số lượng và `coverage: "partial"` khi bị cắt ngắn.

## Diễn giải kết quả

- **`flutter_detected`** là true khi tồn tại bất kỳ `lib/<abi>/libapp.so` nào.
- **`snapshot_hash`** là mã băm bản dựng Dart được quan sát 20 byte sau snapshot magic (`f5 f5 dc dc`) bên trong `libapp.so`.
