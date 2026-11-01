# Hình ảnh UF2 và DfuSe

REA kiểm kê các tệp UF2 (USB Flashing Format của Microsoft) và DfuSe (container cập nhật DFU của ST, `.dfu`) thông qua `inspect-artifact` / `inspect_artifact`, và hiện thực hóa các byte đã giải mã của chúng thông qua `extract-artifact` / `extract_artifact`. UF2 là định dạng kéo thả của bộ nạp khởi động RP2040 và RP2350, của TinyUF2 trên ESP32-S2 và ESP32-S3, và của `esptool merge-bin --format uf2`. Quá trình giải mã là tĩnh và không cần công cụ bên ngoài.

```sh
rea inspect-artifact ./firmware.uf2 --json
rea inspect-artifact ./firmware.dfu --detail summary --evidence_output evidence.json --json
rea extract-artifact ./firmware.uf2 --json
```

Đối với MCP, mở tệp bằng `open_binary`, sau đó gọi `inspect_artifact`. Phân phối tóm tắt giữ lại Bằng chứng hoàn chỉnh; sử dụng `parent_evidence_id` với `inspect_analysis_view` để đọc các vùng và trường đã chọn. `extract_artifact` trả về một `output_root` mới chứa một tệp thô cho mỗi vùng.

## Nhận diện

Một tệp là UF2 chỉ khi 512 byte đầu tiên của nó là một khối hoàn chỉnh với cả ba số magic: `0x0A324655` và `0x9E5D5157` ở đầu và `0x0AB16F30` ở cuối. Một tệp là DfuSe chỉ khi nó bắt đầu bằng `DfuSe` và phiên bản định dạng 1, đồng thời kết thúc bằng hậu tố DFU 1.1a 16 byte: `bcdDFU` `0x011A`, chữ ký `UFD` và `bLength` 16. Tên tệp không quan trọng. Định dạng container là `uf2` và `dfuse`.

## Các vùng và metadata UF2

Mỗi mục là một vùng liền kề địa chỉ của các tải trọng flash chính của một họ, có tên `family-0x<familyID>/0x<start>-0x<end>.bin` với địa chỉ kết thúc bao gồm, ví dụ `family-0xe48bff56/0x10000000-0x1000ffff.bin`. Các khối không có cờ familyID (`0x2000`) nằm dưới `no-family/`. Các họ là các không gian địa chỉ riêng biệt: một tệp đa họ có một vùng cho mỗi họ ngay cả ở cùng các địa chỉ. Các khoảng trống không bao giờ được lấp đầy.

Metadata vùng `format_metadata` (`uf2-region`) ghi lại `family_id` (hoặc `null`), `family_name`, `start_address`, `end_address`, `length`, `block_count`, và `first_file_block`/`last_file_block`.

Metadata gốc `format_metadata` (`uf2`) ghi lại:
- `block_count`, `trailing_bytes`: các khối 512 byte hoàn chỉnh và các byte sau khối hoàn chỉnh cuối cùng;
- `invalid_block_count`: các khối có số magic sai hoặc tải trọng vượt quá 476 byte;
- `flash_block_count`: các khối flash chính hợp lệ, nguồn duy nhất của các byte vùng;
- `family_count`, `family_name_registry`: các familyID riêng biệt giữa các khối flash chính;
- `region_count`, `data_bytes`, `duplicate_bytes`: các vùng, byte đã giải mã duy nhất và byte được ghi lại với giá trị giống hệt nhau.
