# Hình ảnh Flash ESP (ESP Flash Images)

REA đọc bản dump của bộ nhớ flash SPI thuộc họ ESP32, chẳng hạn như đầu ra của `esptool read-flash 0 ALL dump.bin` hoặc `esptool merge-bin`, thông qua `inspect_artifact` và `extract_artifact`. Trình đọc là tĩnh: không cần thiết bị, esptool hay ESP-IDF, và không có mã nào được chạy.

Định dạng container là `esp-flash-image`. Các mục bao gồm vùng bootloader, sector bảng phân vùng và một tệp cho mỗi phân vùng. Bootloader và mỗi phân vùng ứng dụng cũng được giải mã dưới dạng [hình ảnh ESP](esp-image.md), do đó tiêu đề, phân đoạn, tính toàn vẹn và bộ mô tả của chúng xuất hiện trong `format_metadata` gốc.

## Nhận diện

Một tệp là hình ảnh flash khi REA tìm thấy một bảng phân vùng trong cấu trúc mục mà bootloader của ESP-IDF xác minh (`esp_partition_table_verify`): các mục `0x50AA` 32 byte, tối đa một mục MD5 `0xEBEB` trên các mục trước nó và một mục kết thúc trong phạm vi `0xC00` byte. Bảng được định vị theo thứ tự sau:
1. tại `partition_table_offset` của bên gọi, khi được cung cấp;
2. tại vị trí mặc định `0x8000`;
3. bằng cách quét mọi sector flash căn chỉnh theo `0x1000` để tìm bảng có MD5 xác thực.

## Đầu vào của bên gọi

| Đầu vào                   | Ý nghĩa                                                                       |
| ------------------------- | ----------------------------------------------------------------------------- |
| `format: esp-flash-image` | Đọc tệp dưới dạng bản dump flash; nếu không tìm thấy bảng, từ chối nó         |
| `flash_offset`            | Địa chỉ flash (`0x…`) của byte đầu tiên của bản dump; mặc định 0              |
| `partition_table_offset`  | Địa chỉ flash (`0x…`, bội số của `0x1000`) của bảng; vô hiệu hóa việc quét     |
