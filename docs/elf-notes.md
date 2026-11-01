# Bảng ghi chú ELF (ELF Note Tables)

`inspect_elf_notes` / `inspect-elf-notes` trả lời câu hỏi liệu một tệp ảnh ELF32/ELF64 cục bộ đã chọn có khai báo program header `PT_NOTE` hoặc section `SHT_NOTE` hay không, chủ sở hữu và loại ghi chú nào được lưu trữ trong các bảng đó, và những trường GNU build-id hoặc ABI-tag nào mà các ghi chú đó mang theo. Nó đọc tệp mà không cần thực thi và không yêu cầu bộ tháo mã, gói debug hay catalog gói.

```bash
rea inspect-elf-notes /duong/dan/tuyet/doi/toi/program --json
```

CLI phân giải các đường dẫn tương đối dựa trên thư mục làm việc của bên gọi. MCP yêu cầu đường dẫn hệ thống tệp tuyệt đối và bảo toàn lựa chọn chính xác đó:

```json
{
  "name": "inspect_elf_notes",
  "arguments": { "path": "/artifacts/program" }
}
```

## Quan sát (Observations)

Kết quả Bằng chứng (Evidence) nội tuyến chứa đường dẫn hiện vật, SHA-256 và độ dài byte. `tables` liệt kê từng program header hoặc section ghi chú đã khai báo, bao gồm cả tên section khi có bảng chuỗi (string table). Bảng tên section được nén bằng Zlib và Zstandard được giải nén trong ngân sách kích thước tệp đã chọn. Một loại nén không được hỗ trợ sẽ khiến tên section không xác định và thêm một giới hạn; các byte ghi chú vẫn được kiểm tra độc lập. Mỗi mục trong `entries[]` báo cáo chủ sở hữu, loại, SHA-256 của bộ mô tả và bản xem trước dạng hex cho các bộ mô tả từ 64 byte trở xuống.

Tên section cũng giữ là null khi độ lệch của nó nằm ngoài bảng chuỗi đã khai báo hoặc các byte của nó thiếu ký tự kết thúc NUL. Một giới hạn giải thích cho tên bị thiếu đó. Độ bao phủ ghi chú vẫn hoàn chỉnh khi tất cả bảng ghi chú và mục đã khai báo đều được kiểm tra; một tên section không khả dụng không xóa đi những dữ kiện ghi chú đó.

Đối với các ghi chú GNU, `gnu.build_id_hex`, `gnu.abi_tag`, hoặc `gnu.gold_version` được điền khi bộ mô tả khớp với loại đó. Các chủ sở hữu khác giữ `gnu` là null.

`PT_NOTE` và `SHT_NOTE` được kiểm tra độc lập. Cùng một phạm vi tệp có thể xuất hiện hai lần khi cả hai bảng đều khai báo nó.

Danh sách bảng bị thiếu là sự vắng mặt quan sát được của các khai báo đó. Đó không phải là bằng chứng tệp ảnh không bao giờ mang ghi chú trong một container khác hoặc overlay.

## Giới hạn và lỗi (Limits & Failures)

Tệp được chọn được giới hạn ở 64 MiB theo mặc định (tối đa 512 MiB). Tối đa 32 mục ghi chú được kiểm tra theo mặc định trên mọi bảng. Tên ghi chú, tên section và văn bản bộ mô tả GNU được lưu giữ chia sẻ một ngân sách quét dựa trên các byte nguồn cộng với bất kỳ bảng tên section đã giải mã nào, được giới hạn ở `max_file_bytes`. Nguồn thô và bảng đã giải mã cũng phải cùng khớp trong mức trần đó. Nếu việc tiếp nhận thất bại, lỗi sẽ báo cáo `max_file_bytes` bắt buộc; việc nâng mức này lên có thể tiếp nhận bảng đã giải mã lên đến mức tối đa 512 MiB của thao tác. Sau khi được tiếp nhận, việc nâng cài đặt đó không làm tăng ngân sách quét cho cùng tên nguồn và tên đã giải mã. Phạm vi bảng lặp lại được tính lại. Mỗi phạm vi bộ mô tả được băm một lần, và chuỗi NUL phía sau của mỗi phiên bản GNU được quét một lần; không có lần duyệt nào được vượt quá mức trần byte đó. Header ELF bị lỗi, bảng header bị cắt cụt, bảng ghi chú đã khai báo mở rộng vượt quá tệp và các mục ghi chú bị cắt cụt sẽ thất bại mà không có thành công một phần. Đánh số mở rộng ELF (`PN_XNUM`, bao gồm số lượng section được lưu trữ khi `e_shnum` bằng 0) không được hỗ trợ và trả về kết quả mục tiêu không được hỗ trợ thay vì khôi phục đầu vào không hợp lệ.

Bảng tên section Zstandard cũng sử dụng giới hạn cửa sổ giải mã 128 MiB.

Section ghi chú `SHF_COMPRESSED` trả về `unsupported_target` trước khi payload của chúng được diễn giải hoặc giải nén. Header nén của chúng là metadata lưu trữ, không phải header ghi chú. Kết quả hiện tại yêu cầu độ lệch tệp vật lý cho từng ghi chú và bộ mô tả, điều mà các ghi chú nén đã giải mã không thể cung cấp. Sự từ chối này không gắn nhãn bộ lưu trữ ghi chú nén là sai định dạng hoặc cấp phát kích thước giải mã đã khai báo của nó. Bảng tên section được nén vẫn được hỗ trợ trong các giới hạn ở trên. Hãy kiểm tra một bản sao được giải nén riêng biệt khi cần các ghi chú này, giữ lại hiện vật ban đầu để phục vụ việc xác định xuất xứ.

Quy trình làm việc này không thực thi tệp ảnh hoặc xác minh tính duy nhất của GNU build-id dựa trên các catalog gói.
