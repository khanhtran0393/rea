# Tệp Erlang BEAM (Erlang BEAM Files)

`inspect_erlang_beam` / `inspect-erlang-beam` trả lời câu hỏi liệu một tệp cục bộ đã chọn có phải là vùng chứa IFF `FOR1`/`BEAM` hay không, biểu mẫu đó lưu trữ các đoạn (chunks) nào, và các trường atom, export, import, lambda cùng header Code mà các đoạn đó mang theo. Nó đọc tệp mà không cần thực thi bytecode và không yêu cầu cài đặt Erlang hoặc trình giả lập BEAM.

```bash
rea inspect-erlang-beam /duong/dan/tuyet/doi/toi/app.beam --json
```

CLI phân giải các đường dẫn tương đối dựa trên thư mục làm việc của bên gọi. MCP yêu cầu đường dẫn hệ thống tệp tuyệt đối và bảo toàn lựa chọn chính xác đó:

```json
{
  "name": "inspect_erlang_beam",
  "arguments": { "path": "/artifacts/app.beam" }
}
```

## Quan sát (Observations)

Kết quả Bằng chứng (Evidence) nội tuyến chứa đường dẫn hiện vật, SHA-256 và độ dài byte. `chunks` liệt kê từng tên đoạn IFF, phạm vi tệp và SHA-256 của payload. Các tên lặp lại giữ lại các lần xuất hiện đoạn riêng biệt. Các bảng đã giải mã và header Code sử dụng lần xuất hiện cuối cùng của mỗi tên, khớp với bộ nạp của OTP; kho kiểm kê đoạn vẫn để lộ các payload trước đó và phạm vi nguồn của chúng. Kích thước biểu mẫu BEAM giới hạn các đoạn đã phân tích cú pháp. Các byte sau biểu mẫu đã khai báo bị bộ phân tích cú pháp bỏ qua nhưng vẫn là một phần của độ dài byte và SHA-256 của hiện vật đã chọn.

Khi một bảng `AtU8` hoặc `Atom` đã chọn được giải mã, `atoms` là bảng tên dựa trên 1 và `module` là atom 1. Cả bảng trước OTP 28 và bảng nhỏ gọn OTP 28+ (số lượng phủ định, độ dài tên được mã hóa nhỏ gọn) đều được đọc. `exports`, `imports`, và `lambdas` phân giải các chỉ số atom thành những tên đó khi bảng có mặt; các chỉ số chưa được phân giải giữ `function` hoặc `module` là null.

`code` báo cáo tập lệnh của đoạn Code, opcode tối đa, số lượng nhãn, số lượng hàm và phạm vi tệp của luồng lệnh. Các bảng tùy chọn bị thiếu là null, không phải là rỗng ở mọi nơi.

Một `AtU8` 0 byte sẽ quay về bảng `Atom` cũ có sẵn, theo [quy tắc chọn atom của OTP 20.3](https://github.com/erlang/otp/blob/a113f6117fd696ea6f84ed754055b4ec97a7ccb2/erts/emulator/beam/beam_load.c). Một `AtU8` không rỗng giữ lại ưu tiên UTF-8, bao gồm cả số lượng atom bằng 0 rõ ràng; dữ liệu không rỗng bị lỗi sẽ thất bại thay vì quay về dự phòng. Một đoạn atom rỗng không có bảng thay thế vẫn thất bại ở header số lượng bắt buộc của nó. Các đoạn `FunT` tùy chọn bị thiếu và 0 byte tạo ra `lambdas: null`, trong khi số lượng hàng bằng 0 4 byte tạo ra `lambdas: []`. Điều này tuân theo [quy tắc phân tích cú pháp lambda kích thước dương của OTP 29](https://github.com/erlang/otp/blob/550d7b7898706c7822362c42e7b93120c1d1f29a/erts/emulator/beam/beam_file.c). Payload rỗng giữ lại phạm vi và SHA-256 của chúng trong kiểm kê đoạn. Các quy tắc này áp dụng cho lần xuất hiện cuối cùng của mỗi tên đoạn; một lần xuất hiện cuối cùng rỗng không làm sống lại payload trước đó của tên đó. Các bảng không rỗng bị cắt cụt vẫn là các lỗi định dạng.

## Chọn dữ kiện được lưu giữ (Select Retained Facts)

Đầu ra hoàn chỉnh vẫn là mặc định. Yêu cầu một bản tóm tắt với chế độ xem `beam` để đọc một vài dữ kiện trong lần gọi đầu tiên trong khi vẫn giữ lại cuộc kiểm tra hoàn chỉnh:

```json
{
  "name": "inspect_erlang_beam",
  "arguments": {
    "path": "/artifacts/app.beam",
    "detail": "summary",
    "view": {
      "kind": "beam",
      "collection": "exports",
      "names": ["module_info"],
      "fields": ["function_index", "function", "arity", "label"],
      "limit": 20
    }
  }
}
```

`normalized_result.page.items` chứa các hàng khớp. `source_index` dựa trên 0 trong bộ sưu tập ban đầu và được giới hạn phạm vi theo `parent_evidence_id` và bộ sưu tập đó. Nó không thay thế `index` atom dựa trên 1 ban đầu hoặc `index` của chính lambda. Mọi tên hàm khớp đều tồn tại, bao gồm cả các arity khác nhau. Tên là chuỗi hiển thị được lưu giữ chính xác; chỉ mục thô vẫn khả dụng khi tên chưa được phân giải hoặc tên hiển thị xung đột.

Các bộ sưu tập gồm `atoms`, `exports`, `imports`, `lambdas`, và `chunks`. `names` chọn tên atom/đoạn hoặc tên hàm; imports hỗ trợ thêm các `modules` chính xác. Các bộ lọc giao nhau. `indices` chọn các chỉ mục bộ sưu tập ban đầu; `fields` chọn các trường riêng của bộ sưu tập đó. Mảng bộ chọn rỗng không khớp với mục nào và `fields: []` chỉ trả về `source_index`. Offset/limit phân trang các hàng khớp theo thứ tự ban đầu (mặc định 0/64). Các trang không được lọc cắt lát bộ sưu tập được lưu giữ; các truy vấn không giải mã lại tệp.

Đối với `chunks`, `names` khớp với thẻ đoạn IFF 4 ký tự (ví dụ `"AtU8"`, `"Code"`, `"ExpT"`); các `fields` khả dụng là `index`, `name`, `location`, và `payload`. Đối với `lambdas`, `names` khớp với tên hàm đã giải quyết từ bảng atom; tên `null` (chỉ số atom chưa phân giải) không bao giờ được chọn bởi bộ lọc tên. Đối với `atoms`, `names` khớp với chuỗi tên atom đã giải mã.

`beam` giữ lại mô-đun, ranh giới biểu mẫu/Code, độ bao phủ nguồn ban đầu và số lượng bảng. Số đếm null nghĩa là không có bảng nào được giải mã; 0 nghĩa là một bảng rỗng đã giải mã. Bảng bị thiếu và tên atom chưa phân giải vẫn rõ ràng, và một trang trống không xác lập sự vắng mặt runtime. Độ bao phủ trang tách biệt với các đoạn được kiểm tra và ngân sách bộ phân tích cú pháp của nguồn.

Phân phối tóm tắt CLI yêu cầu một đích cho gói Bằng chứng hoàn chỉnh:

```sh
rea inspect-erlang-beam /artifacts/app.beam --detail summary \
  --view '{"kind":"beam","collection":"exports","names":["module_info"],"fields":["function","arity"]}' \
  --evidence_output /reports/app-beam.evidence.json --json
rea inspect-analysis-view '{"source":{"kind":"evidence-file","path":"/reports/app-beam.evidence.json"},"view":{"kind":"beam","collection":"imports","modules":["erlang"]}}' --json
```

Gói giữ lại báo cáo hoàn chỉnh. Các truy vấn MCP tiếp theo có thể sử dụng `source: {kind: "retained-evidence", evidence_id: parent_evidence_id}` hoặc tệp Bằng chứng. Chế độ xem ngoại tuyến và được lưu giữ vẫn hoạt động sau khi tệp BEAM gốc bị xóa. Cung cấp `view` yêu cầu `detail: "summary"`; bỏ qua nó chỉ trả về bản tóm tắt cấu trúc. Việc lựa chọn chỉ thay đổi phân phối, không thay đổi phân tích cú pháp hoặc độ bao phủ và không thực thi bytecode nào.

## Giới hạn và lỗi (Limits & Failures)

Tệp được chọn được giới hạn ở 64 MiB theo mặc định (tối đa 512 MiB). Tối đa 64 đoạn và 4096 atom được kiểm tra theo mặc định, và tối đa 65536 hàng export, import, và lambda cùng nhau. Header `FOR1`/`BEAM` bị lỗi, các đoạn bị cắt cụt và tên `AtU8` không phải là UTF-8 hợp lệ sẽ thất bại mà không có thành công một phần.

Quy trình làm việc này không thực thi bytecode, không giải nén ký tự `LitT`, hoặc giải mã payload thuật ngữ bên ngoài `Attr`/`CInf`. Độ bao phủ hoàn chỉnh mô tả báo cáo bảng và đoạn tĩnh này; nó không xác lập rằng một runtime Erlang có thể nạp hoặc thực thi mô-đun.
