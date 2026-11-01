# Snapshot cơ sở dữ liệu SQLite

`inspect_sqlite_database` / `inspect-sqlite-database` kiểm tra một snapshot cơ sở dữ liệu SQLite cục bộ được chọn rõ ràng mà không cần mục tiêu phân tích native đang hoạt động. Sử dụng lệnh này để hiểu lưu trữ ứng dụng, cấu hình hoặc bằng chứng cơ sở dữ liệu được giữ lại. Không cần cài đặt SQLite riêng biệt.

Thao tác này yêu cầu Node.js 24.x >=24.15 hoặc 26+ với API native `DatabaseSync.limits`. `rea capabilities` báo cáo tính khả dụng của `inspect_sqlite_database` và yêu cầu runtime mà không cần mở cơ sở dữ liệu.

```sh
rea capabilities --json
rea inspect-sqlite-database ./application.sqlite --json
rea inspect-sqlite-database ./application.sqlite --table settings --column key --column value --row-limit 20 --json
```

```json
{
  "name": "inspect_sqlite_database",
  "arguments": {
    "path": "/artifacts/application.sqlite",
    "table": "settings",
    "columns": ["key", "value"],
    "row_limit": 20
  }
}
```

Đường dẫn MCP phải là đường dẫn hệ thống tệp tuyệt đối trên máy chủ hiện tại. Hãy chọn một snapshot tĩnh (quiescent): dừng tiến trình ghi hoặc lấy một cặp database/WAL nhất quán bằng quy trình sao lưu của ứng dụng trước khi kiểm tra.

## Bằng chứng trả về

Nếu không có lựa chọn bảng, `schema_scope` mặc định là `inventory`: metadata bảng thu gọn cùng tên index, view và trigger, không có khai báo SQL hoặc chi tiết cột. Chọn một bảng sẽ mặc định là `selected_table`, mở rộng bảng đó cùng các index và trigger của nó. Đặt rõ ràng `schema_scope: "database"` (hoặc CLI `--schema-scope database`) sẽ mở rộng tất cả các khai báo đã lưu trữ và các cột được hỗ trợ.

Tên bảng chính xác `sqlite_schema` cũng là một nguồn hàng chỉ đọc được hỗ trợ.

Các hàng là tùy chọn. `table` chọn một tên bảng thông thường chính xác. `columns` (CLI lặp lại `--column`) chọn tên cột hiển thị chính xác theo thứ tự yêu cầu. Các ô không được chọn sẽ không được đánh giá hay chiếu. `row_limit` yêu cầu `table`, mặc định là 20.

Số nguyên SQLite là các chuỗi thập phân để mọi giá trị 64-bit có dấu tồn tại qua quá trình tuần tự hóa JSON. Các giá trị thực giữ nguyên số của chúng; `"Infinity"`, `"-Infinity"` và `"-0"` bảo toàn rõ ràng các giá trị vô cực và số không âm qua JSON. Văn bản trả về chuỗi đã giải mã cùng với các byte của nó dưới dạng hex.
