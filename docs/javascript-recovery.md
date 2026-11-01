# Khôi phục mã nguồn JavaScript (JavaScript Source Recovery)

`recover_javascript_sources` / `recover-javascript-sources` dẫn xuất các module có thể đọc được từ các tệp hoặc bundle JavaScript UTF-8 được chọn rõ ràng. Sử dụng lệnh này trên các tệp được trả về bởi `export_web_scripts` hoặc các script cục bộ khác. Thao tác không bao giờ thực thi mã ứng dụng được chọn hoặc khôi phục.

## Engine do bên gọi cung cấp

REA chấp nhận Wakaru **^1.13.0** trên Linux x64. Trình phân tích cú pháp báo cáo được xác minh với Wakaru **1.14.0**. Một bản phát hành khác trong phạm vi đó được chấp nhận, phiên bản quan sát được báo cáo và một giới hạn ghi lại rằng trình phân tích cú pháp chưa được xác minh đối với bản phát hành đó. Wakaru thay đổi `--version`, `--json` và `provenance.json` chỉ bằng cách bổ sung trong một phiên bản chính ([chính sách tương thích](https://github.com/pionxzh/wakaru/blob/v1.14.0/docs/cli.md#machine-readable-output-compatibility)), do đó trình phân tích cú pháp giữ các trường báo cáo unknown và trạng thái module được báo cáo.

Đặt `REA_WAKARU_COMMAND` thành đường dẫn tuyệt đối của tệp thực thi do bên gọi cung cấp. Adapter sử dụng `prlimit` của util-linux (mặc định `/usr/bin/prlimit`; ghi đè bằng đường dẫn tuyệt đối `REA_JAVASCRIPT_PRLIMIT_COMMAND`). REA không cài đặt công cụ nào trong hai công cụ này. Cấu hình engine chỉ được kiểm tra khi yêu cầu khôi phục; các tính năng khác của REA vẫn khả dụng mà không cần cấu hình này.

Mã nguồn upstream chưa sửa đổi được lưu giữ trong `third_party/wakaru`, ghim theo `8219bf5016a063d7fb215101a8c45d5959ec733f` (v1.14.0, Apache-2.0). Kết quả báo cáo đường dẫn chính xác của tệp nhị phân đã thực thi, SHA-256 và phiên bản quan sát được. Bản sửa đổi nguồn đã kiểm toán tách biệt khỏi `executed_source_revision: null`: một chuỗi phiên bản không thể chứng minh mã nguồn nào đã tạo ra tệp nhị phân do bên gọi cung cấp.

### Cài đặt Wakaru

Sử dụng tệp nhị phân phát hành chính thức, cùng tệp mà CI xác minh:

```sh
curl -fL https://github.com/pionxzh/wakaru/releases/download/v1.14.0/wakaru-linux-x64.tar.gz | tar -xz wakaru
export REA_WAKARU_COMMAND="$PWD/wakaru"
```

Hoặc cài đặt từ npm. Trỏ tới tệp nhị phân nền tảng, không phải shim Node.js trong `node_modules/.bin`:

```sh
npm install wakaru@1.14.0
export REA_WAKARU_COMMAND="$(realpath node_modules/@wakaru/cli-linux-x64/wakaru)"
```

Cả hai đều cho cùng một tệp nhị phân (SHA-256 `664fa06782a61eda7ed15885134b52525fd565dd77607e642062ceb6cd927c38`).

## Khôi phục và phân tích

```sh
rea recover-javascript-sources /tmp/web-scripts/app.js /tmp/recovered-app --json
rea analyze-javascript-application /tmp/recovered-app/modules --json

# Khôi phục các đoạn code-split liên quan cùng nhau; lặp lại --additional-path.
rea recover-javascript-sources /tmp/web-scripts/main.js /tmp/recovered-chunks   --additional-path /tmp/web-scripts/runtime.js   --additional-path /tmp/web-scripts/shared.js   --additional-path /tmp/web-scripts/checkout.chunk.js --json
```

Đầu vào MCP:

```json
{
  "path": "/tmp/web-scripts/main.js",
  "additional_paths": [
    "/tmp/web-scripts/runtime.js",
    "/tmp/web-scripts/shared.js",
    "/tmp/web-scripts/checkout.chunk.js"
  ],
  "output_directory": "/tmp/recovered-chunks"
}
```

Thư mục đầu ra phải chưa tồn tại, là đường dẫn tuyệt đối và có thư mục cha hiện có. Mỗi nguồn được tạo snapshot riêng biệt trong một workspace riêng tư. Nhiều đầu vào giữ lại bố cục thư mục tương đối của chúng và được truyền cùng nhau đến một lần gọi Wakaru duy nhất, cho phép các ID module liên chunk được giải quyết. Đầu vào phải riêng biệt; `additional_paths` chấp nhận tối đa 16 tệp bổ sung. Engine chỉ ghi vào thư mục staging của riêng nó. Đường dẫn module được báo cáo, danh tính đầu vào, dải byte UTF-8, tệp thông thường và biểu diễn map phát ra được xác thực trước khi xuất bản. REA ghi các byte đã xác minh vào một cây đầu ra thuộc quyền sở hữu độc quyền và hoàn nguyên nó khi thất bại. Nó cũng kiểm tra xem nguồn gốc và byte engine không thay đổi trước khi commit. Đầu ra hiện có không bao giờ bị ghi đè.

Nội dung xuất bản bao gồm:
- `inputs/bundle.js`: snapshot gốc chính xác cho một đầu vào đơn lẻ.
- `modules/`: các module đã khôi phục.
- `provenance.json`: báo cáo nguồn gốc khôi phục.
