# Khôi phục metadata NativeAOT tùy chọn

REA có một trình phân tích cú pháp snapshot chỉ đọc tích hợp cho các mục tiêu PE x64 với metadata .NET 8 RTR 9.1. Nó phân tích cú pháp snapshot phiên Ghidra chính xác đã tiếp nhận và sử dụng lớp phủ trong bộ nhớ cho metadata được bù nước (rehydrated); nó không xác định DataType, nhãn, hàm hoặc byte bộ nhớ của Ghidra. Ngữ nghĩa bù nước được điều chỉnh từ [Washi1337/ghidra-nativeaot](https://github.com/Washi1337/ghidra-nativeaot) tại `effeb734fc570c32650f88b159608979dc7b423e` (MIT), được kiểm tra chéo với `MethodTable.h` của .NET tại runtime commit `a3b2d40328be0be3f8dd43f0e5cc925b8827b044` (MIT).

Adapter Ghidra headless tùy chọn vẫn khả dụng cho các chú thích cơ sở dữ liệu và khôi phục rộng hơn.

Trình phân tích cú pháp hiện xử lý PE32+ x64 và RTR 9.1, bao gồm xử lý DEHYDRATED_DATA có giới hạn, các mối quan hệ MethodTable, các slot ảo và chuỗi đóng băng (frozen strings). Các phiên bản RTR và kiến trúc PE khác được báo cáo là không được hỗ trợ hoặc chỉ hỗ trợ một phần.

Trên máy chủ Windows, Windows Ghidra P0 chạy trình phân tích cú pháp mà không cần JAR. Metadata, các mối quan hệ MethodTable, slot ảo và chuỗi đóng băng đến từ báo cáo của trình phân tích cú pháp, và `analyze_function` dịch ngược các mục tiêu slot. Các công cụ đọc cơ sở dữ liệu Ghidra không thấy các byte được bù nước, vì chỉ có adapter tùy chọn mới ghi chúng vào cơ sở dữ liệu và Windows Ghidra P0 không có quyền thay đổi cơ sở dữ liệu. Một assembly PE/CLI hoặc ReadyToRun nên sử dụng `inspect_managed_artifact`; nó không phải là NativeAOT.

## Xây dựng và cấu hình

Tự cung cấp bản cài đặt Ghidra/JDK được hỗ trợ:

```sh
git submodule update --init third_party/ghidra-nativeaot
npm run build:ghidra:nativeaot
export REA_GHIDRA_NATIVEAOT_JAR="$PWD/_reference/nativeaot-integration/extension/rea-ghidra-nativeaot.jar"
```

Bản dựng yêu cầu bản checkout sạch đã ghim, biên dịch Java upstream chưa sửa đổi cộng với facade của REA và ghi JAR cùng build manifest vào thư mục bị bỏ qua `_reference/`.

REA cam kết SHA-256 thực tế của JAR vào profile phân tích, sao chép các byte giống hệt vào runtime thuộc sở hữu, kiểm tra lại mã băm trước khi nạp và đóng classloader/project khi đóng phiên.

## Sử dụng các công cụ hiện có

Mở tệp thực thi với provider `ghidra`. Đối với PE x64, `inspect_native_load_image` trả về `observations.metadata_recovery` nội dòng từ snapshot nguồn đã thu thập, bao gồm tiêu đề RTR, địa chỉ MethodTable, chuỗi đóng băng, danh tính lớp phủ dẫn xuất, cùng số lượng đã kiểm tra/khôi phục.

Truyền địa chỉ MethodTable được trả về tới `inspect_native_data_type`. Truy vấn PE trực tiếp cũng dẫn xuất metadata snapshot từ bắt tay phiên Ghidra đã xác thực.

Đường dẫn CLI tích hợp sử dụng địa chỉ MethodTable được trả về bởi trình phân tích cú pháp hoặc quan sát khác:

```sh
rea inspect-native-load-image ./NativeAotFixture --provider ghidra --json
rea inspect-native-data-type ./NativeAotFixture --address 0xMETHODTABLE_ADDRESS --provider ghidra --json
rea function ./NativeAotFixture 0xADDRESS --provider ghidra --json
```

Đường dẫn danh mục `/NativeAOT/...` được tạo thuộc về adapter headless tùy chọn. Chỉ sử dụng `--type /NativeAOT/MethodTables/Class_ADDRESS_MT` khi adapter đó được cấu hình và đã tạo DataType Ghidra tương ứng.
