# Hình ảnh ESP (ESP Images)

REA đọc các hình ảnh firmware của họ ESP32 thông qua `inspect-artifact` / `inspect_artifact` và `extract-artifact` / `extract_artifact`. Các bản tải xuống OTA, phân vùng ứng dụng và bootloader giai đoạn hai đều là các hình ảnh ESP. Việc đọc là tĩnh: không cần esptool, thiết bị hay quyền truy cập flash, và không có mã nào được chạy hay dịch ngược.

```sh
rea inspect-artifact ./firmware.bin --json
rea inspect-artifact ./ota.bin --detail summary --evidence_output evidence.json --json
rea extract-artifact ./firmware.bin --json
```

Đối với MCP, mở tệp bằng `open_binary`, sau đó gọi `inspect_artifact`. `extract_artifact` ghi một tệp cho mỗi phân đoạn và cho mỗi khối chữ ký.

Dữ kiện bố cục tuân theo ESP-IDF v6.1 và esptool v5.6.0.

## Nhận diện

Định dạng container là `esp-image`. Một tệp là một hình ảnh ESP khi các byte của nó hiển thị tất cả các đặc điểm sau:
- magic `0xE9` tại độ lệch 0 và có từ 1 đến 16 phân đoạn;
- mã chế độ flash và tốc độ flash mà các enum của ESP-IDF định nghĩa;
- byte `hash_appended` là 0 hoặc 1, như esptool yêu cầu;
- khi tệp chứa tiêu đề phân đoạn đầu tiên, độ dài phân đoạn là bội số của 4.

## Dữ kiện hình ảnh

Metadata gốc `format_metadata` có `kind: "esp-image"`:
- `header`: mọi trường `esp_image_header_t` với tên của ESP-IDF;
- `chip`: `target`, `isa` từ `chip_id`;
- `checksum`: `offset`, `stored`, `computed` và `status`;
- `appended_sha256`: mã băm do `hash_appended` thêm vào;
- `descriptor`: các trường `esp_app_desc_t` hoặc `esp_bootloader_desc_t`.
