# Đồ thị ứng dụng JavaScript (JavaScript Application Graph)

JavaScript Application Graph là mô hình miền trung lập với provider của REA để kết nối các byte ứng dụng được phân phối, cấu trúc JavaScript, ranh giới tiến trình Electron, các quan sát trình duyệt thụ động và các add-on native. Đây là một hợp đồng dữ liệu chuẩn hóa, lâu bền; nó không phải là một bộ trích xuất hay một công cụ CLI/MCP mới.

Triển khai hoàn toàn thuần túy và không có tác dụng phụ. Nó xác thực các dữ kiện do bên gọi cung cấp, dẫn xuất các mã định danh ngữ nghĩa, thực thi tính toàn vẹn của đồ thị và tuần tự hóa các đồ thị đã xác minh dưới dạng JSON chuẩn RFC 8785. Nó không thực hiện I/O hệ thống tệp hay CDP, không đánh giá JavaScript và không có loại node dành riêng cho ứng dụng.

## Ranh giới các lớp

Đồ thị bổ sung cho các mô hình REA hiện có thay vì thay thế chúng:

| Mô hình hiện có              | Trách nhiệm                                                                            | Sử dụng trong Application Graph                             |
| ---------------------------- | -------------------------------------------------------------------------------------- | ---------------------------------------------------------- |
| Đồ thị artifact              | Container định địa chỉ nội dung, các lần xuất hiện byte và ranh giới trích xuất         | Cung cấp danh tính artifact và đường dẫn chính xác          |
| Phân tích web bundle         | Các quan sát AST, chunk, route, endpoint và source-map có giới hạn                      | Cung cấp các node và mối quan hệ tĩnh                       |
| Quan sát Browser/Electron    | Metadata CDP thụ động đã được ủy quyền và các bản thu thập runtime                      | Cung cấp các thể hiện runtime và quan hệ `observed_as`      |
| Phân tích Native             | Các hàm nhị phân, biểu tượng và tham chiếu theo provider                               | Cung cấp các quan sát xuất/add-on native                    |
| JavaScript Application Graph | Danh tính thực thể liên lớp, các quan hệ mang bằng chứng và các cam kết chuẩn hóa       | Bảo toàn mô hình ứng dụng kết hợp                          |

Dịch vụ [tái cấu trúc artifact JavaScript](javascript-artifact-reconstruction.md) chiếu các thư mục/ASAR, package, bundle, source-map, BrowserWindow, preload, contextBridge, IPC, tiến trình tiện ích và liên kết native phía JavaScript vào đồ thị này. Dịch vụ [điều hòa tĩnh/runtime](javascript-runtime-reconciliation.md) bổ sung các mục tiêu, khung, script, worker và các cạnh `observed_as` suy diễn.

## Bản ghi được lưu trữ

Một đồ thị được lưu trữ có cấu trúc cấp cao nhất như sau:

```text
schema: "JavaScriptApplicationGraph"
graph_id: jag_<sha256 của ngữ nghĩa đồ thị chuẩn hóa>
root_node_ids: jag_node_<sha256>[]
nodes: các thực thể ứng dụng với một hoặc nhiều quan sát
edges: các mối quan hệ có hướng mang bằng chứng
coverage: complete, partial, unknown, hoặc unavailable
limitations: các ràng buộc rõ ràng trên toàn đồ thị
```

Việc phân tích cú pháp từ chối các trường không xác định, thứ tự không chuẩn, ID trùng lặp, cam kết ngữ nghĩa cũ, thiếu gốc, điểm cuối lơ lửng, cạnh tự thân và các cạnh `changed_from` giữa các loại node khác nhau.

Từ vựng quan hệ có hướng bao gồm `contains`, `loads`, `imports`, `maps_to`, `exposes`, `sends`, `invokes`, `handles`, `calls`, `accesses`, `persists_to`, `observed_as`, và `changed_from`.
