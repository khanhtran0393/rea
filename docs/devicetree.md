# Kiểm tra cây thiết bị phẳng (Flattened Devicetree - DTB)

`inspect_devicetree` / `rea inspect-devicetree` đọc tệp DTB cục bộ mà không cần engine, thực thi mục tiêu hay công cụ bên ngoài. Lệnh hỗ trợ FDT phiên bản 16 và 17; các phiên bản khác trả về `unsupported_target`. Blob nhận diện bị hỏng trả về `invalid_request`, và vượt quá ngân sách tài nguyên trả về `resource_constraint` mà không thành công một phần.

```bash
rea inspect-devicetree /artifacts/board.dtb --json
rea inspect-devicetree /artifacts/board.dtb --detail summary --evidence_output /artifacts/board-evidence.json --json
```

Bằng chứng hoàn chỉnh bao gồm SHA-256 của artifact, các trường tiêu đề đã xác thực, các vùng nhớ dành riêng 64-bit dưới dạng chuỗi thập lục phân chính xác, và cây node phẳng với đường dẫn cha rõ ràng. Mỗi thuộc tính giữ lại độ lệch tệp, độ dài byte và tối đa 256 byte thô dạng hex. `truncated` chỉ bổ nghĩa cho bản xem trước; độ bao phủ phân tích cú pháp bao gồm toàn bộ cây đã khai báo. Phiên bản 16 không có kích thước cấu trúc được khai báo, do đó `header.size_dt_struct` là null.

Các trường dẫn xuất mang `confidence: derived`: danh sách chuỗi ASCII tiêu chuẩn, phandle vô hướng và số lượng cell, cùng các tuple `reg` sử dụng `#address-cells` và `#size-cells` của cha (mặc định 2 và 1). Các thành phần 0-cell là null (bỏ qua), khác biệt với số 0 được mã hóa. Số lượng cell trên 4, số lượng không hợp lệ và tuple không đầy đủ vẫn không được giải mã kèm theo giới hạn. Địa chỉ là cục bộ trên bus; không có bản dịch `ranges` nào được thử nghiệm. Chuỗi không hợp lệ hoặc mơ hồ vẫn giữ nguyên dạng thô, không phỏng đoán.

Bản tóm tắt được dán nhãn là derived và bao gồm compatible/model gốc, bootargs/stdout-path và các bí danh (aliases) đã chọn. Một bí danh không thể giải mã sẽ giữ lại tên của nó với đường dẫn null; nó không bị âm thầm coi là vắng mặt. Các tham chiếu `interrupt-parent`, `remote-endpoint` và `msi-parent` đơn cell được giải quyết thông qua các khai báo phandle. Mục tiêu bị thiếu hoặc trùng lặp có đường dẫn mục tiêu là null. Node có `phandle` và `linux,phandle` không khớp nhau sẽ thêm một giới hạn, và các tham chiếu đến một trong hai giá trị sẽ không được giải quyết. `phandle` hiện diện nhưng không phải là một cell cũng thêm một giới hạn; `linux,phandle` không thể giải quyết node đó khi khai báo chuẩn bị lỗi. Các mảng tham chiếu đặc thù liên kết (binding) và overlay không được giải quyết; danh sách tham chiếu trống không chứng minh sự vắng mặt của các liên kết phần cứng.

MCP `detail: summary` giữ lại Bằng chứng hoàn chỉnh trong phiên làm việc. Lần theo `parent_evidence_id` của nó với `inspect_analysis_view`:

```json
{
  "source": { "kind": "retained-evidence", "evidence_id": "ev:<digest>" },
  "view": {
    "kind": "devicetree",
    "collection": "properties",
    "path": "/serial@1000",
    "offset": 0,
    "limit": 32
  }
}
```

Các collection bao gồm `nodes` (danh tính và số lượng thuộc tính), `properties` (yêu cầu đường dẫn node chính xác), `reservations`, và `references`. Các trang sử dụng các dữ kiện đã phân tích được lưu giữ và không mở lại tệp. Mỗi trang mang ngữ cảnh kích thước cố định (artifact, header, độ bao phủ và số lượng) thay vì các chuỗi tóm tắt, do đó giá trị `bootargs` lớn không lặp lại trên mọi trang. Chỉ mục nguồn thuộc phạm vi collection đã chọn và Bằng chứng cha; chỉ mục thuộc tính thuộc phạm vi node của chúng.

| Ngân sách đầu vào | Mặc định | Mức trần |
| ----------------- | -------: | -------: |
| `max_file_bytes`  |    4 MiB |   16 MiB |
| `max_nodes`       |    8.192 |   65.536 |
| `max_properties`  |   32.768 |  262.144 |
| `max_depth`       |       64 |      256 |

Tên, đường dẫn, giá trị đã giải mã và chi phí hàng dùng chung mức phụ cấp biểu diễn cố định 32 MiB, được tính trước khi mở rộng. Nó không có ghi đè lúc runtime.

Số lượng đặt trước (reservation) cũng sử dụng `max_properties`. Các cờ CLI sử dụng dạng kebab-case, ví dụ `--max-nodes 16384`. Các trang khung nhìn chứa tối đa 256 hàng.

## Xác minh

Các bài kiểm tra tổng hợp bao gồm v16/v17, tiêu đề/khối/tên/token lỗi, giới hạn tài nguyên, các giá trị dẫn xuất, và phân phối CLI/MCP bao gồm các trang được lưu giữ sau khi xóa tệp đầu vào. Luồng thực tế tùy chọn yêu cầu **dtc 1.8.1**:

```bash
npm run build:cached
node scripts/verify/firmware/devicetree.mjs
```

Nó biên dịch một DTS với reserved-memory, chosen, aliases và phandles, so sánh đường dẫn node của REA với `dtc -I dtb -O dts`, và kiểm tra báo cáo dẫn xuất. `REA_DTC_PATH` chọn tệp thực thi xác minh; nó không phải là cấu hình runtime.

Luồng dtc 1.8.1 đã chạy trên macOS arm64 với Node 26.5.0. Bảy đường dẫn node của nó khớp với cây đã dịch ngược. Báo cáo bao gồm `model: "REA DTB verification"`, `bootargs: "console=ttyS0"`, bí danh `serial0 -> /serial@1000`, đặt trước `0x80000000 + 0x1000`, và một tham chiếu đã giải quyết từ `/consumer` đến `/serial@1000`. Fixture remote-endpoint tối giản có chủ đích tạo ra cảnh báo liên kết đồ thị của dtc; luồng này xác minh cấu trúc FDT và các phép dẫn xuất thuộc tính đã chọn, không phải tính hợp lệ của liên kết phần cứng.
