# Kiểm tra container iNES cổ điển (Classic iNES)

REA nhận diện định dạng iNES cổ điển thông qua chữ ký byte `NES` + `0x1a`, độc lập với phần mở rộng tệp. Quá trình kiểm tra container chỉ đọc (read-only) này sử dụng các hàm đọc tệp Node.js khả chuyển; không yêu cầu Ghidra hay trình giả lập.

```bash
rea inspect-artifact ./cartridge.nes --json
rea extract-artifact ./cartridge.nes --json
```

Thông qua MCP, gọi `open_binary` với đường dẫn cục bộ, sau đó gọi `inspect_artifact`. Sử dụng `extract_artifact` để trích xuất các phân vùng vào thư mục mới được đặt tên theo kết quả trả về. Bên gọi sở hữu thư mục trích xuất thành công đó.

## Tiêu đề (Header) và các phân vùng tệp

Quá trình kiểm tra trả về các quan sát `container-header` và `reconstruction` nội dòng. Bằng chứng kiểm kê (Evidence inventory) duy nhất dưới `substeps[].evidence.normalized_result` chứa đồ thị hoàn chỉnh và đối tượng `ines`:

- `ines.header`: `header_hex` 16-byte gốc, số lượng đơn vị tệp PRG/CHR, mapper, các cờ thô 6/7, cờ trainer và battery, cùng chế độ mirroring đã khai báo.
- `occurrences[]`: đường dẫn của từng phân vùng, `source_location.offset` và `source_location.length` trong container gốc. Lần theo `artifact_id` đến mã băm SHA-256 và số lượng byte quan sát được của node.
- `ines.reconstruction`: danh sách đường dẫn phân vùng theo thứ tự, tổng số byte ghép nối, SHA-256 và `verified: true`. REA tích lũy mã băm ghép nối trong cùng một lượt quét phân vùng theo thứ tự vật lý sinh ra băm node, sau đó so sánh với mã băm toàn bộ tệp gốc. Độ tươi mới (freshness) của nguồn cuối cùng cũng được kiểm tra. Không có tệp tái tạo nào được ghi trong quá trình kiểm tra.

Các phân vùng bao gồm `header.bin`, `trainer.bin` tùy chọn (512 byte), từ `prg/000.bin` trở đi (16 KiB mỗi khối), từ `chr/000.bin` trở đi (8 KiB mỗi khối), và `trailing.bin` tùy chọn. Mọi byte đầu vào đều được giữ lại, bao gồm các trường header không dùng và dữ liệu đuôi chưa được diễn giải. Không có đơn vị CHR-ROM nào sẽ không tạo phân vùng CHR; REA không tự tạo nội dung CHR-RAM.

Sau khi trích xuất, ghép nối các tệp đã tạo theo `ines.reconstruction.region_order` từ bản kiểm kê. Các byte này tái tạo chính xác tệp đầu vào. Thứ tự đường dẫn trong đồ thị là thứ tự từ điển, không phải thứ tự tệp vật lý; hãy sử dụng thứ tự phân vùng rõ ràng hoặc độ lệch nguồn.

## Giới hạn hỗ trợ

Phần nâng cấp này hỗ trợ iNES cổ điển với tiêu đề 16 byte hoàn chỉnh, số lượng PRG khác không, tải trọng trainer/PRG/CHR đã khai báo đầy đủ và các byte dành riêng 12–15 bằng 0. Các tiêu đề cũ không sạch (dirty legacy headers) bị từ chối vì các bit mapper trên bị mơ hồ; REA không tự ý sửa chúng. Các chữ ký NES 2.0 và UNIF, các biến thể tiêu đề không được hỗ trợ và tải trọng bị cắt ngắn đều bị từ chối rõ ràng. Tệp bất kỳ mang đuôi `.nes` sẽ không được chấp nhận nếu không đúng định dạng iNES.

Các đơn vị PRG/CHR mô tả lưu trữ tệp, không phải kích thước bank mapper hay cửa sổ CPU. Số mapper và các cờ là khai báo tiêu đề, không phải bằng chứng phần cứng cartridge hay hành vi runtime. Chuyển bank (bank switching), dịch ngược 6502/65816, phiên giả lập và diễn giải dữ liệu đuôi đặc thù cho máy console nằm ngoài phạm vi này. Phân tích nhị phân thô bằng Ghidra là một luồng công việc riêng. Bản container này giải quyết cột mốc đầu tiên của [#2072](https://github.com/morluto/rea/issues/2072); vấn đề [#1641](https://github.com/morluto/rea/issues/1641) liên quan theo dõi phân tích native 6502/65816 với địa chỉ nạp rõ ràng và giới hạn của provider. Các đơn vị PRG được trích xuất vẫn cần cửa sổ CPU do bên gọi chọn trước khi phân tích mã tĩnh; độ lệch tệp vật lý của chúng không xác lập các địa chỉ đó.

Các fixture byte thuộc mã nguồn xác minh độ lệch và mã băm gốc, quá trình trích xuất và tái tạo chính xác từng byte, tính tương đương giữa CLI/stdio MCP, đầu vào lỗi/không hỗ trợ, phục hồi khi chuyển mục tiêu thất bại, hủy bỏ và định danh nguồn. Các kiểm tra này không xác lập hành vi giả lập hay độ tương thích với mọi ROM trong lịch sử. Các kiểm tra chấp nhận CLI/stdio MCP khả chuyển và danh tính nguồn chạy trong CI Linux và macOS. Các quy trình này cũng đã được thực thi trên Windows x64 native.

Tài liệu tham khảo định dạng: [iNES](https://www.nesdev.org/wiki/INES) và [NES 2.0](https://www.nesdev.org/wiki/NES_2.0).
