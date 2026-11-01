# Kiểm kê tĩnh gói ứng dụng HarmonyOS

REA kiểm kê các gói ứng dụng HarmonyOS thông qua trình đọc artifact ZIP được gia cố dùng chung trong `src/artifacts/`, sau đó chiếu bản kiểm kê đã xác thực vào một đồ thị ứng dụng không thực thi. Đây là bước tăng dần đầu tiên trong lộ trình HarmonyOS (#1431): chỉ nhận diện gói và kiểm kê thành phần.

Các định dạng gốc được nhận diện:

- `.hap` — module ứng dụng có thể cài đặt
- `.hsp` — gói dùng chung runtime (runtime shared package)
- `.app` — App Pack phân phối (các phần tử con HAP/HSP được liệt kê theo đường dẫn). Phân loại App Pack chỉ áp dụng cho các đầu vào gốc có magic ZIP đã xác thực; một thành viên ZIP lồng nhau kết thúc bằng `.app` bên trong một gói khác, chẳng hạn như IPA, giữ phân loại `zip` thông thường.

Các thư viện dùng chung HarmonyOS `.har` cố ý không được phân loại theo phần mở rộng: `.har` cũng đặt tên cho HTTP Archive JSON, do đó một kho lưu trữ dùng chung Harmony yêu cầu các byte ZIP đã xác thực và được hoãn lại với giới hạn rõ ràng.

## Cách sử dụng

Không yêu cầu SDK, toolchain hoặc engine bên ngoài. Kiểm kê gói và chiếu đồ thị; CLI và MCP sử dụng cùng một quy trình làm việc và trả về Bằng chứng gắn với artifact nội dòng:

```sh
rea inspect-artifact /path/to/Example.hap --format json > inspection.json
node --input-type=module -e 'import { readFileSync } from "node:fs"; console.log(JSON.stringify({ inventory_evidence: [JSON.parse(readFileSync("inspection.json", "utf8"))] }));' > projection-input.json
rea project-harmony-application-graph projection-input.json --format json
```

Công cụ MCP `project_harmony_application_graph` nhận cùng đầu vào `inventory_evidence`. Phép chiếu báo cáo:

- các đường dẫn thành phần chính xác, ID artifact và mã băm (manifest, Ark bytecode, tài nguyên, thư viện native, JavaScript, chữ ký)
- gợi ý mô hình đóng gói: Stage chỉ được khẳng định từ mục `module.json` viết hoa-thường chính xác bên ngoài đường dẫn tài nguyên đã phân loại, FA chỉ từ `config.json` bên ngoài đường dẫn tài nguyên đã phân loại, nếu không là `unknown`
- tài nguyên FA dưới `assets/<module>/resources/` cũng như Stage `resources/`
- các đường dẫn thành phần không khả dụng với ID và mã băm unknown, bao gồm các mục đã mã hóa; những mục này làm cho độ bao phủ phép chiếu là một phần
- đường dẫn gói con của App Pack mà không kiểm kê đệ quy
- gợi ý họ runtime (`ark`, `native`, `javascript`) từ đường dẫn và định dạng
- giả thuyết cầu nối N-API dựa trên đường dẫn giữa bytecode và thư viện native `libs/<abi>`

Một manifest không khả dụng vẫn có thể cung cấp gợi ý mô hình dựa trên đường dẫn; danh tính nội dung của nó vẫn là unknown. Tên manifest tuân theo tên thành viên ZIP có phân biệt chữ hoa chữ thường.

Sự xuất hiện của tệp và lát cắt cung cấp đường dẫn thành phần và gói con; cây thư mục và liên kết mềm (symlink) giữ nguyên danh tính của chúng trong kiểm kê nguồn mà không bị phân loại là tệp mã. Các gợi ý runtime Ark và JavaScript tóm tắt danh sách bytecode và JavaScript đã phân loại; các phần mở rộng nguồn được hỗ trợ chia sẻ cùng một quy tắc phân loại như các hàng thành phần của chúng.

Mọi xác nhận chỉ là sự hiện diện của đường dẫn. Ngữ nghĩa manifest (danh tính module, abilities, entry points, quyền, phụ thuộc), nội dung và phiên bản Ark bytecode, giải quyết tài nguyên, xác minh chữ ký, và ranh giới native quan sát được nằm ngoài phạm vi của bước này và ở lại trong mảng `limitations` của phép chiếu cho đến khi có provider HarmonyOS chuyên dụng. REA từ chối xử lý (fail-closed) đối với các gói có kiểm kê không thể xác thực.

## Luồng xác minh

`npm run fixtures:harmony` tải bản phát hành HAP 1.0.1 được ghim SHA-256 từ [kho lưu trữ nguồn harmony-next-pipeline](https://github.com/ohosvscode/harmony-next-pipeline) theo giấy phép MIT vào thư mục bị bỏ qua `_reference/harmony-integration/`. Việc ghim ghi lại bản sửa đổi nguồn, ID tài sản phát hành, kích thước và mã băm. Artifact đã tải xuống không được phân phối lại; giấy phép thành phần đi kèm chưa được kiểm toán độc lập.

```sh
npm run fixtures:harmony
REA_HARMONY_TEST_HAP=_reference/harmony-integration/entry-default-unsigned.hap   npm run verify:harmony
```

Luồng này kiểm tra CLI đã biên dịch và stdio MCP thực tế kết nối SDK: tính xác thực của `inspect_artifact`, tính tất định của phép chiếu, gợi ý đường dẫn mô hình Stage, tập hợp thành phần manifest/bytecode/tài nguyên chính xác, tập hợp native và cầu nối rỗng, độ bao phủ kiểm kê hoàn chỉnh, và tính tương đương giữa CLI/MCP. Nó kiểm tra xem fixture này không có mục ký META-INF; điều đó không xác lập việc gói có được ký hay không.
