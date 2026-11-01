# Hình ảnh Intel HEX và Motorola S-record

REA kiểm kê các tệp lập trình Intel HEX (I8HEX, I16HEX, I32HEX) và Motorola S-record (S19, S28, S37) thông qua `inspect-artifact` / `inspect_artifact`, và hiện thực hóa các byte đã giải mã của chúng thông qua `extract-artifact` / `extract_artifact`. Quá trình giải mã là tĩnh và không cần công cụ bên ngoài.

```sh
rea inspect-artifact ./firmware.hex --json
rea inspect-artifact ./firmware.s37 --detail summary --evidence_output evidence.json --json
rea extract-artifact ./firmware.hex --json
```

Đối với MCP, mở tệp bằng `open_binary`, sau đó gọi `inspect_artifact`. Phân phối tóm tắt giữ lại Bằng chứng hoàn chỉnh; sử dụng `parent_evidence_id` với `inspect_analysis_view` để đọc các vùng và trường đã chọn. `extract_artifact` trả về một `output_root` mới chứa một tệp thô cho mỗi vùng.

## Nhận diện

Một tệp chỉ được nhận diện khi dòng đầu tiên của nó, bắt đầu từ byte 0, là một bản ghi hoàn toàn hợp lệ: ký tự bắt đầu chính xác, chữ số hex, số lượng byte, checksum, loại bản ghi đã biết và độ dài cố định. Tên tệp không quan trọng. Byte-order mark, dòng trống đầu trang hoặc bản ghi đầu tiên không hợp lệ sẽ khiến tệp không được nhận diện, do đó nó rơi vào phân loại `file` thông thường. Nhận diện không bao giờ tìm kiếm một bản ghi ở phần sau của tệp. Định dạng container là `intel-hex` và `srec`.

## Các vùng và metadata

Mỗi mục kiểm kê là một vùng byte đã giải mã liền kề địa chỉ, được đặt tên là `0x<start>-0x<end>.bin` với địa chỉ kết thúc bao gồm 8 chữ số hex viết thường, ví dụ `0x08000000-0x0800001f.bin`. Các vùng được liệt kê theo thứ tự địa chỉ tăng dần. Các bản ghi mất thứ tự được sắp xếp vào các vùng; khoảng trống phân tách các vùng và không bao giờ được lấp đầy.

Metadata vùng `format_metadata` (`hex-record-region`) ghi lại `start_address`, `end_address`, `length`, `record_count`, và `first_line`/`last_line`: dải dòng tính từ 1 của các bản ghi dữ liệu đóng góp.

Metadata gốc `format_metadata` (`hex-records`) ghi lại:

| Trường                                            | Ý nghĩa                                                                           |
| ------------------------------------------------- | --------------------------------------------------------------------------------- |
| `encoding`, `variant`                             | `intel-hex` hoặc `srec`; biến thể bắt nguồn từ các loại bản ghi đã thấy, hoặc `mixed` |
| `line_count`, `record_count`, `data_record_count` | Số dòng đã đọc, bản ghi không trống và bản ghi dữ liệu                            |
| `data_bytes`, `duplicate_bytes`, `region_count`   | Byte đã giải mã duy nhất, byte được ghi lại với giá trị giống hệt và số lượng vùng |
| `lowest_address`, `highest_address`               | Ranh giới địa chỉ của byte đã giải mã, hoặc `null` khi không có dữ liệu           |
| `termination`                                     | Liệu Intel `01` hoặc S-record `S7`/`S8`/`S9` có hiện diện hay không               |
| `start_address`, `start_address_record`           | Intel `05` EIP hoặc địa chỉ `S7`/`S8`/`S9`, cùng loại bản ghi của nó              |
| `start_segment_cs`, `start_segment_ip`            | Cặp CS:IP Intel `03`                                                              |
| `header_hex`, `header_text`                       | Dữ liệu S-record `S0` dưới dạng hex, và dưới dạng văn bản chỉ khi mọi byte là ASCII in được |
| `declared_data_record_count`                      | Số lượng `S5`/`S6` cuối cùng, được xác thực đối chiếu với các bản ghi dữ liệu trước đó |

## Giới hạn

Tệp phải có kích thước tối đa 128 MiB. Mỗi byte dữ liệu cần hai chữ số hex, do đó bộ giải mã dành riêng tối đa một nửa kích thước tệp cho các byte đã giải mã trước khi đọc và chỉ lưu trữ các địa chỉ đã ghi. Bộ giải mã giữ lại tối đa 65.536 lượt bản ghi không liền kề địa chỉ trước khi hợp nhất chúng thành các vùng.
