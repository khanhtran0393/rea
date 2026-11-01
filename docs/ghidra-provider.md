# Ngữ nghĩa provider Ghidra (Ghidra Provider Semantics)

REA nạp một mục tiêu vào cơ sở dữ liệu Ghidra tạm thời. Linux/macOS hỗ trợ phân tích chỉ đọc và chỉnh sửa tên hàm/chú thích entry nguyên tử; Windows P0 là chỉ đọc. Các điều khiển GUI và thay đổi cơ sở dữ liệu liên tục không khả dụng. Byte thực thi không thay đổi.

Sau khi bắt tay khởi động đã được xác thực, các đầu vào bị từ chối (`invalid_request`, `not_found`, `ambiguous`, hoặc tên hàm không hợp lệ) sử dụng `diagnostics.diagnostic_scope: "request"`.

Lựa chọn provider và danh tính profile tuân theo [Hợp đồng MCP](mcp-contracts.md#identity-and-discovery). Một provider sâu duy nhất vẫn bị ràng buộc với mục tiêu; lỗi runtime không bao giờ tự động chọn provider khác.

## Mức trần tải trọng yêu cầu

Các yêu cầu Ghidra sử dụng `REA_BRIDGE_MAX_REQUEST_BYTES`: mặc định 16 MiB, với phạm vi cấu hình từ 7.464 đến 536.870.887 byte.

## Ngữ nghĩa kiểm kê được tiếp nhận

| Mối quan tâm     | Hợp đồng Ghidra                                                                                                                                                                                |
| ---------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Định danh chương trình | Một lần nạp `analyzeHeadless` tạo ra chính xác một Chương trình; `list_documents` do đó trả về chính xác một tên.                                                                               |
| Địa chỉ          | Bộ nhớ mặc định sử dụng thập lục phân `0x` viết thường. Không gian không mặc định và bên ngoài sử dụng `<percent-encoded-space>:0x<hex>`.                                                        |
| Biểu tượng       | `list_names` bao gồm bộ nhớ mang địa chỉ và các biểu tượng bên ngoài, bao gồm cả biểu tượng động.                                                                                              |
| Hàm (Procedures) | Cả hàm không phải bên ngoài và hàm bên ngoài đều được liệt kê. Thunk cục bộ tách biệt khỏi mục tiêu đã giải quyết của nó.                                                                      |
| Chuỗi (Strings)  | Chỉ `Data` chuỗi do Ghidra xác định mới được quan sát. Các mục báo cáo bảng mã, độ dài byte và liệu có thiếu ký tự kết thúc null bắt buộc hay không.                                            |
| Bộ nhớ           | Địa chỉ kết thúc khối bộ nhớ là độc quyền. Read/write/execute, khởi tạo, lớp phủ, không gian địa chỉ và image base là các quan sát trực tiếp của Ghidra.                                        |
