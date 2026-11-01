# Thiết bị đo và phân tích JEB (JEB Instrumentation & Analysis)

REA cung cấp khả năng kiểm tra dự án, đơn vị (unit) và dịch ngược được hỗ trợ bởi JEB thông qua CLI và MCP, được phục vụ bởi chính giao diện MCP của JEB. JEB là một engine do người dùng tự cung cấp (bring-your-own): REA không bao giờ tự cài đặt, khởi chạy hoặc mở cơ sở dữ liệu dự án của nó.

## Engine tự cung cấp

Khởi động một client JEB phục vụ MCP và trỏ REA tới nó:

- Endpoint mặc định: `http://127.0.0.1:8425/mcp`. Ghi đè bằng `REA_JEB_MCP_URL` (URL `http`/`https` tuyệt đối với host là `127.0.0.1` hoặc `[::1]`; chuyển hướng, host từ xa, thông tin xác thực, chuỗi truy vấn và đoạn mã đều bị từ chối thay vì âm thầm loại bỏ).
- Trong client GUI JEB, khởi động máy chủ MCP từ menu _File_ (hoặc thông qua trợ lý VIBRE).
- Ở chế độ headless, bản dựng Linux CE chính thức `5.48.0.202610071539` hỗ trợ `-c --mcp`. Từ thư mục cài đặt JEB, chọn JDK 21 hoặc mới hơn qua `JAVA_HOME`, chọn mục tiêu và liên kết rõ ràng với loopback:

  ```sh
  "$JAVA_HOME/bin/java" -jar bin/app/jeb.jar -c --mcp --interface=127.0.0.1     --port=8425 --infile=/absolute/path/to/target
  ```

  Hoàn thành EULA lần đầu sử dụng của nhà cung cấp và kích hoạt mã bản quyền khi được nhắc.

- Đường dẫn lập trình được tài liệu hóa của JEB (`JebMcpServerInstance` từ JEB API) vẫn là một giải pháp thay thế.

Để đăng ký thiết lập, hãy đặt `REA_JEB_MCP_URL` trong môi trường tiến trình REA trước khi chạy `rea setup`. Kế hoạch thiết lập đã phê duyệt giữ lại URL chính xác đã chọn sau khi xác thực cú pháp của nó.

Catalog discovery và mỗi `tools/call` chia sẻ `REA_JEB_MCP_TIMEOUT_MS`. Đặt nó trong môi trường khởi chạy của REA thành một số nguyên từ 1 đến 2147483647 mili giây, ví dụ `REA_JEB_MCP_TIMEOUT_MS=300000` cho ngân sách 5 phút. Nếu bỏ qua, mặc định là 60000 ms của MCP client SDK (`@modelcontextprotocol/client` 2.3.1).

## Các thao tác (Operations)

| Công cụ MCP / Lệnh CLI                      | Mục đích                                                                                                                        |
| ------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| `inspect_jeb_client` / `inspect-jeb-client` | Phiên bản engine, chế độ GUI hoặc headless, thời gian khởi động                                                                 |
| `open_jeb_project` / `open-jeb-project`     | Mở hoặc tạo một dự án từ artifact hoặc `.jdb2` trên máy chủ engine; trả về mã băm SHA-256 tệp đầu vào và các unit cấp cao nhất |
| `list_jeb_units` / `list-jeb-units`         | Phân trang các unit theo đường dẫn và kiểu ổn định (`filter`, `parent-unit-path`, `index`, `count`)                              |
| `decompile_jeb_item` / `decompile-jeb-item` | Dịch ngược một kiểu hoặc phương thức thành mã giả của engine                                                                    |

Cả bốn thao tác đều bảo toàn các byte tệp mục tiêu. `open_jeb_project` chọn hoặc tạo dự án phía engine; các truy vấn phân tích có thể điền các phân tích đã tính toán vào trong đó. Bằng chứng ghi lại endpoint, các tham số do bên gọi chọn và các mã băm sẵn có do engine báo cáo.

## Ngữ nghĩa (Semantics)

- Đường dẫn trong `open_jeb_project` được giải quyết bởi tiến trình JEB, không phải bởi REA; REA không yêu cầu artifact phải tồn tại cục bộ.
- Engine giới hạn các trang `list_units` ở 100; một trang đầy đủ được báo cáo là `coverage: "partial"` vì có thể tồn tại nhiều unit hơn.
- Khi `decompile_jeb_item` bỏ qua `unit_path`, engine ngầm chọn unit mã đầu tiên của dự án; REA báo cáo `unit_path: null` và một giới hạn thay vì đoán unit hiệu dụng.
- `item_address` là tham chiếu mục mã JEB, chẳng hạn như giá trị được trả về bởi `list_code_items` của engine. Bảo toàn biểu diễn đó và chọn unit mã của nó một cách rõ ràng.
- Các từ chối hợp lệ của engine (phong bì `success: false`) trả về `execution_failure` với thông báo của engine khi khả dụng.
