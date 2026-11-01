# Khớp hàm native qua các bản dựng (Native Function Matching)

`match_native_functions` ghép nối hai bản kiểm kê Bằng chứng `list_procedures` hiện có. Chọn `list_procedures` `detail: "summary"` để giữ lại các hàng nguồn hoàn chỉnh mà không cần chuyển toàn bộ bản kiểm kê. `parent_evidence_id` của nó là tham chiếu mà việc khớp sử dụng.

Các mục tiêu có thể được mở tuần tự; việc so khớp không bao giờ tự mở một mục tiêu, đọc lại tệp của nó, thực thi nó hoặc khởi động một provider. Hãy bảo toàn từng bản kiểm kê trước khi đóng mục tiêu của nó, hoặc import bundle Bằng chứng của nó sau. `close_binary` xóa các bản ghi phiên. Truyền Bằng chứng hoặc các tham chiếu phiên chính xác dưới dạng `left` và `right`:

```json
{
  "left": { "kind": "retained-evidence", "evidence_id": "ev_<left digest>" },
  "right": { "kind": "retained-evidence", "evidence_id": "ev_<right digest>" }
}
```

Tên chính xác duy nhất có nguồn gốc biểu tượng quan sát được ghép nối trước. Các hàng kiểm kê hoặc Bằng chứng `list_names` tùy chọn xác định tên động/mặc định cần loại trừ. Nguồn gốc unknown hoặc mâu thuẫn không xác lập sự tương ứng về tên; các byte thân hàm độc lập hoàn chỉnh vẫn đủ điều kiện. Tên chính xác xác lập sự tương ứng về tên, không phải mã không đổi hay danh tính nguồn gốc. Ghidra xuất bản các dữ kiện biểu tượng của nó; Hopper, IDA và các hồ sơ cũ hơn giữ lại nguồn gốc unknown khi không quan sát thấy các dữ kiện như vậy.

Đối với các ứng viên chưa khớp, tùy chọn cung cấp `left_observations` và `right_observations`. `procedure_info` hoặc hồ sơ `analyze_function` hiện có cung cấp các dải thân hàm bao gồm chính xác; `read_bytes` cung cấp các byte bộ nhớ. Phản hồi tóm tắt `analyze_function` của MCP không phải là hồ sơ: hãy truyền `parent_evidence_id` của nó dưới dạng tham chiếu `retained-evidence`. Một khớp byte yêu cầu mọi byte thân hàm, tổng kích thước bằng nhau, độ dài dải có thứ tự bằng nhau, mã băm SHA-256 duy nhất và cùng một kiến trúc đã biết.

Mặc định `detail: "summary"` giữ lại Bằng chứng khớp hoàn chỉnh trong MCP và trả về số lượng, cả danh tính và độ bao phủ. Truy vấn `parent_evidence_id` của nó:

```json
{
  "source": {
    "kind": "retained-evidence",
    "evidence_id": "ev_<matching digest>"
  },
  "view": {
    "kind": "function-matches",
    "collection": "functions",
    "states": ["matched"],
    "sides": ["left"],
    "fields": ["name", "body_sha256"],
    "offset": 0,
    "limit": 20
  }
}
```

`collection: "matches"` trả về danh tính quan hệ, trạng thái, cơ sở và số lượng thành viên. Danh sách thành viên lớn được bỏ qua theo mặc định; hãy chọn rõ ràng các cột `left`/`right` hoặc phân trang từng `functions`.
