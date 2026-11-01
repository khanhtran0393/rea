# Hình ảnh U-Boot FIT (Flattened Image Tree)

REA kiểm kê các tệp U-Boot Flattened Image Tree (FIT, `.itb`) thông qua `inspect-artifact` / `inspect_artifact`, và hiện thực hóa các byte được lưu trữ của từng hình ảnh thông qua `extract-artifact` / `extract_artifact`. Quá trình giải mã là tĩnh và không cần công cụ bên ngoài.

```sh
rea inspect-artifact ./boot.itb --json
rea inspect-artifact ./boot.itb --detail summary --evidence_output evidence.json --json
rea extract-artifact ./boot.itb --json
```

Đối với MCP, mở tệp bằng `open_binary`, sau đó gọi `inspect_artifact`. Phân phối tóm tắt giữ lại Bằng chứng hoàn chỉnh; sử dụng `parent_evidence_id` với `inspect_analysis_view` để đọc các hình ảnh và trường đã chọn. `extract_artifact` trả về một `output_root` mới chứa một tệp thô cho mỗi hình ảnh.

## Nhận diện

FIT là một flattened devicetree (FDT v16 hoặc v17) với node cấp cao nhất `/images`. Việc nhận diện duyệt khối cấu trúc chỉ trong 4 KiB đầu tiên của tệp; tên tệp không quan trọng. Cây thiết bị có node `/images` bắt đầu sau 4 KiB, cây thiết bị không có `/images`, và bất kỳ phiên bản FDT nào khác đều đi theo luồng devicetree thông thường. Định dạng container là `fit`.

## Hình ảnh và metadata

Mỗi phần tử con của `/images` là một mục kiểm kê có tên `images/<unit-name>.bin`, được liệt kê theo thứ tự devicetree. Các byte của nó đến từ chính xác một nguồn dữ liệu:

| Nguồn           | Byte được lưu trữ                                                                       |
| --------------- | --------------------------------------------------------------------------------------- |
| `data`          | Giá trị thuộc tính nhúng                                                                 |
| `data-offset`   | `data-size` byte tại `data-offset` qua phần cuối của devicetree, căn chỉnh theo 4 (`mkimage -E`) |
| `data-position` | `data-size` byte tại `data-position` tính từ đầu tệp (`mkimage -p`)                      |

Metadata hình ảnh `format_metadata` (`fit-image`) ghi lại `index`, `name`, `description`, `type`, `arch`, `os`, `compression`, `load` và `entry`, `data_source`, `declared_offset` thô cho dữ liệu bên ngoài, `image_data`, `cipher_present`, `cipher_algo`, `hash_algorithms`, `hashes_verified`, `hashes_unverified`, `signature_count`, `signatures_unreadable` và `signature_algorithms`.

Byte hình ảnh được báo cáo theo cách lưu trữ. Hình ảnh bị nén không được giải nén và hình ảnh mã hóa không được giải mã: các mục báo cáo `encrypted: false` cùng khai báo mật mã trong `cipher_present` và thuật toán trong `cipher_algo`.

## Mã băm và chữ ký

Các node con mã băm là các phần tử con có tên bắt đầu bằng `hash`. Bản kiểm kê xác minh `crc32` (4 byte, big-endian), `md5`, `sha1`, `sha256`, `sha384` và `sha512` trên các byte được lưu trữ. Tất cả các mã băm của một hình ảnh được tính toán trong một lượt truyền dữ liệu duy nhất trong khi bản kiểm kê được chuẩn bị, do đó một sự không khớp sẽ làm thất bại toàn bộ bản kiểm kê. Chữ ký không bao giờ được xác minh tự động.
