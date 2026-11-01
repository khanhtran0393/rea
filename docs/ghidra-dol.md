# Phân tích GameCube DOL qua Ghidra

Luồng này nạp các hình ảnh PowerPC GameCube/Wii DOL 32-bit big-endian thông qua Ghidra. Nó không thực thi hình ảnh. DOL là định dạng không có tiêu đề chuẩn (headerless) và không có magic number, do đó REA không bao giờ tự phát hiện nó: bên gọi phải chọn nó một cách rõ ràng.

Chỉ có cửa sổ MEM1 được lưu trong bộ nhớ cache `0x80000000..0x81800000` được tiếp nhận. Wii MEM2 và các bí danh không được lưu trong bộ nhớ cache nằm ngoài luồng này.

Chưa hỗ trợ: điểm neo dữ liệu nhỏ (small-data anchors: `r2`/`r13`), REL, RPX, và hỗ trợ IDA hoặc Hopper. Xem [Chưa được hỗ trợ](#chua-duoc-ho-tro).

## Chọn định dạng

CLI:

```bash
rea inspect /absolute/path/to/main.dol --target-format gamecube-dol --provider ghidra --json
rea function /absolute/path/to/main.dol 0x80003130 --target-format gamecube-dol --provider ghidra --json
```

MCP, với công cụ `open_binary` hiện có:

```json
{
  "path": "/absolute/path/to/main.dol",
  "provider_id": "ghidra",
  "format": "gamecube-dol"
}
```

Kiến trúc bắt nguồn từ tiêu đề DOL (`powerpc`). Không truyền `architecture` cùng với `gamecube-dol`; nó sẽ bị từ chối. `entry_point` là tùy chọn, và nếu được cung cấp thì nó phải bằng với điểm nhập trong tiêu đề.

Việc bỏ qua `format` sẽ không bao giờ nạp tệp dưới dạng DOL. Nhận diện tự động không chạy đối với DOL.

## Thao tác import thực hiện những gì

REA đọc tiêu đề DOL 256 byte khi tiếp nhận. Nó kiểm tra dải tệp và địa chỉ nạp của từng phần (section). Nó ghi lại các phần BSS/dữ liệu chồng chéo và tính toán chính xác các khoảng trống BSS chưa được khởi tạo một lần. Các tiêu đề bị lỗi sẽ bị từ chối trước khi Ghidra khởi động.

Ghidra sau đó chạy theo hai bước:

1. `BinaryLoader` tiêu chuẩn ánh xạ toàn bộ tệp tại địa chỉ cơ sở `0x0`, với `-processor` và `-cspec` được lấy từ lựa chọn ngôn ngữ bên dưới.
2. Script tiền xử lý `bridge/ghidra/ReaGhidraPrepareDol.java` đọc một tệp section mà REA đã ghi từ tiêu đề đã phân tích, với số lượng tập hợp được đánh phiên bản. Nó tạo một khối đã khởi tạo cho mỗi phần text và data được sử dụng tại địa chỉ nạp của nó, với các byte tệp từ độ lệch tệp của phần đó. Khối toàn bộ tệp bị xóa trước khi các khối section được tạo. Text là read-execute và data là read-write. Nó thiết lập điểm nhập và gắn nhãn cho entry là `__start`.

Các phần mà tiêu đề đánh dấu là không sử dụng sẽ không được ánh xạ. Tiêu đề 256 byte và bất kỳ byte nào giữa các phần cũng không được ánh xạ.

## BSS

Một tệp DOL có một dải BSS duy nhất và nó không phải lúc nào cũng tách biệt khỏi các phần dữ liệu. Trình phân tích cú pháp miền tính toán các phần của BSS mà không có phần dữ liệu nào bao phủ. Cả script tiền xử lý và chứng thực hình ảnh tải đều sử dụng các dải đã phân tích đó. Script tiền xử lý chỉ tạo các khối read-write chưa khởi tạo trong các khoảng trống đó. Nơi BSS chồng chéo với một slot dữ liệu, slot đó sẽ giữ lại các byte tệp của nó. Các chỉ mục slot chồng chéo được báo cáo bởi trình phân tích cú pháp.

## Ngôn ngữ ràng buộc

Ngôn ngữ mặc định là `PowerPC:BE:32:default` với thông số trình biên dịch `default`, đi kèm sẵn với Ghidra. Chứng thực hình ảnh nạp kiểm tra ngôn ngữ mà Ghidra báo cáo, không chỉ ngôn ngữ được yêu cầu.

### Ghi đè Gekko/Broadway tùy chọn

`REA_GHIDRA_LANGUAGE_ID=PowerPC:BE:32:Gekko_Broadway` chọn biến thể Gekko/Broadway. Ngôn ngữ này không có trong Ghidra gốc. Nó đến từ tiện ích mở rộng GameCubeLoader của Cuyler, được cấp phép theo Apache-2.0. Tiện ích mở rộng cung cấp `ppc_gekko_broadway.ldefs` trong thư mục `data/languages` của nó và phải được cài đặt trong thư mục `Ghidra/Extensions` của bản cài đặt Ghidra. Luồng mặc định không bao giờ yêu cầu tiện ích này.

Bất kỳ ID ngôn ngữ nào khác phải bắt đầu bằng `PowerPC:BE:32:`. Các ngôn ngữ khác bị từ chối với ràng buộc cụ thể.

## Giới hạn paired-single

Ngôn ngữ `PowerPC:BE:32:default` gốc không giải mã các lệnh paired-single mà Gekko và Broadway bổ sung. Các từ lệnh đó giải mã thành các lệnh hỏng hoặc không xác định. Sử dụng ghi đè Gekko để giải mã chúng.

## Xác minh

Fixture thuộc mã nguồn được xây dựng từ `tests/conformance/c/ppc.c`, `tests/conformance/c/dol.c`, `tests/conformance/ghidra/ppc-probe.S` và linker script `tests/conformance/ghidra/dol-fixture.ld`. Nó được chuyển đổi thành DOL bởi `tests/conformance/ghidra/dol-fixture.mjs`. Phần text của nó ở `0x80003100`, và dải BSS duy nhất của nó chồng chéo với data slot 1 và để lại một khoảng trống. Fixture này không bao giờ được thực thi. Hình ảnh trò chơi GameCube hoặc Wii thực tế không bao giờ được sử dụng trong kiểm thử.
