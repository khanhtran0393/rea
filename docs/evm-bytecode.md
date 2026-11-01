# Bytecode EVM ngoại tuyến (Offline EVM Bytecode)

`inspect_evm_interface` / `inspect-evm-interface` suy diễn các ứng viên giao diện từ một vật mang bytecode cục bộ được chọn rõ ràng. Khả năng này mang tính tổng quát: điều tra hợp đồng, tính tương thích, kiểm toán và các bài thi CTF sử dụng cùng một thành phần khả chuyển. Lệnh không yêu cầu mục tiêu nhị phân native đang hoạt động.

```sh
rea inspect-evm-interface ./runtime.hex hex --json
rea inspect-evm-interface ./runtime.bin raw --json
```

```json
{
  "name": "inspect_evm_interface",
  "arguments": { "path": "/artifacts/runtime.hex", "encoding": "hex" }
}
```

Xác minh thực tế ban đầu bao gồm bytecode legacy phẳng trên Linux x64 và EVMole 0.9.3 đi kèm chưa sửa đổi. Các container EF00 kiểu EOF trả về `unsupported_target`. Máy chủ phải cung cấp `prlimit` của util-linux, thông thường là `/usr/bin/prlimit`; `REA_EVM_PRLIMIT_COMMAND` có thể chọn một tệp thực thi tuyệt đối khác. REA không cài đặt công cụ hệ thống và không thay đổi cấu hình người dùng.

## Ngữ nghĩa Bằng chứng

- `encoding` là bắt buộc: byte thô (`raw`) hoặc hex UTF-8 (`hex`), với tiền tố `0x` tùy chọn và khoảng trắng ASCII bên ngoài. Hex lẻ, dấu phân cách bên trong, BOM, khoảng trắng không phải ASCII và UTF-8 không hợp lệ đều bị từ chối. Bytecode trống là hợp lệ và có độ bao phủ unknown.
- SHA-256 của vật mang xác định tệp gốc chính xác. SHA-256 byte đã giải mã xác định các byte EVM thực tế đã chọn. Cả hai mã băm đều không phải là mã băm Keccak của Ethereum.
- Toàn bộ bytecode, độ lệch hàm upstream và biểu diễn của trình tạo ban đầu được trả về nội dòng theo mặc định. Độ lệch là các vị trí byte trong bytecode đã giải mã.
- `bytecode.metadata` diễn giải độc lập một map gốc CBOR kết thúc hợp lý từ các byte đã chọn. Số nguyên sử dụng chuỗi thập phân; chuỗi byte sử dụng hex và các khóa vô hướng giữ lại các kiểu của chúng.
- Bộ chọn (selectors), chuỗi đối số và tính khả biến trạng thái (state mutability) là **suy diễn**. Tên/chữ ký hàm không được tra cứu. Mã được tối ưu hóa có thể tạo ra các ứng viên khả biến khác nhau; các kiểu được khôi phục không phải là ABI nguồn đã xác minh.
- Phân tích là ngoại tuyến: không thực thi runtime mục tiêu, không tra cứu chuỗi/RPC/explorer ngầm, không thao tác ví, không phát sóng giao dịch hoặc yêu cầu mạng.

## Phân phối giao diện đã chọn và các khung nhìn được giữ lại

Trên máy chủ MCP giữ lại Bằng chứng, hãy chọn `detail: "summary"` khi câu hỏi cần mọi hàm được suy diễn và danh tính/độ không chắc chắn của nó, mà không cần văn bản bytecode hoặc stdout thô trùng lặp:

```json
{
  "name": "inspect_evm_interface",
  "arguments": {
    "path": "/artifacts/runtime.hex",
    "encoding": "hex",
    "detail": "summary"
  }
}
```

Sử dụng `normalized_result.parent_evidence_id` với `inspect_analysis_view` kiểu nguồn `retained-evidence`. Kiểu view EVM `evm` phân trang `functions` theo thứ tự hàng ban đầu, hoặc `bytecode_hex`, `stdout`, `stderr` theo đơn vị mã UTF-16.
