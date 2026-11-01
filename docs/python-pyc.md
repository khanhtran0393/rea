# Kiểm tra tệp CPython pyc (CPython pyc Inspection)

`inspect_python_pyc` / `inspect-python-pyc` báo cáo định dạng và phiên bản CPython được hỗ trợ do magic của một tệp `.pyc` cục bộ đã chọn khai báo, cách header 16 byte của nó ghi lại danh tính nguồn và những đối tượng mã lồng nhau nào mà đồ thị marshal đã giải mã của nó chứa. Nó đọc tệp mà không cần chạy bytecode và không yêu cầu cài đặt Python, mục tiêu nhị phân đang hoạt động, Hopper, Ghidra hay IDA.

```bash
rea inspect-python-pyc /duong/dan/tuyet/doi/toi/module.pyc --json
```

CLI phân giải các đường dẫn tương đối dựa trên thư mục làm việc của bên gọi. MCP yêu cầu đường dẫn hệ thống tệp tuyệt đối và bảo toàn lựa chọn chính xác đó:

```json
{
  "name": "inspect_python_pyc",
  "arguments": { "path": "/artifacts/module.pyc" }
}
```

## Các đối tượng mã đã chọn (Selected Code Objects)

Đầu ra hoàn chỉnh vẫn là mặc định. Để đọc các đối tượng mã đã chọn từ một báo cáo lớn, hãy yêu cầu MCP `detail: "summary"`. Bản tóm tắt chứa danh tính hiện vật/header, độ bao phủ nguồn và số lượng giới hạn; `normalized_result.parent_evidence_id` của nó xác định báo cáo hoàn chỉnh được lưu giữ. Sử dụng ID cha đó với `inspect_analysis_view` và:

```json
{
  "kind": "python-pyc",
  "collection": "code_objects",
  "names": ["parse_args", "parse_known_args"],
  "fields": [
    "name",
    "qualname",
    "filename",
    "firstlineno",
    "bytecode",
    "nested_code"
  ],
  "offset": 0,
  "limit": 64
}
```

Đây là trường `view` của yêu cầu. Để nhận các dữ kiện đã chọn này trong lệnh gọi đầu tiên, hãy đưa trực tiếp cùng một `view` vào `inspect_python_pyc` cùng với `detail: "summary"`. Các trang sau sử dụng ID cha được trả về với `inspect_analysis_view`. `view` yêu cầu phân phối tóm tắt; phân phối hoàn chỉnh từ chối một view thay vì bỏ qua nó. `names` khớp với các chuỗi hiển thị được lưu giữ và trả về mọi đối tượng có cùng tên. `indices` chọn các chỉ mục `code_objects` ban đầu; việc kết hợp names và indices sẽ giao cắt các bộ chọn đó. Bộ chọn bị bỏ qua bao gồm tất cả các hàng, trong khi `[]` không chọn hàng nào. Mỗi hàng luôn bao gồm `source_index`, được giới hạn phạm vi theo Bằng chứng cha của nó; `fields: []` chỉ trả về chỉ mục đó. Lọc bảo toàn thứ tự nguồn và phân trang đếm các hàng khớp. `pyc.coverage` của view bảo toàn độ bao phủ nguồn độc lập với độ bao phủ trang. Các view bảo toàn chỉ mục `nested_code` và ngữ nghĩa hằng số mã của chúng.

Phân phối tóm tắt CLI yêu cầu một đích bền vững cho Bằng chứng hoàn chỉnh:

```bash
rea inspect-python-pyc module.pyc --detail summary --evidence_output module.evidence.json --json
```

Thêm `--view '{"kind":"python-pyc","collection":"code_objects","names":["parse_args"],"fields":["name","bytecode"]}'` để chọn các dữ kiện trong lệnh gọi CLI ban đầu đó.

`summary` của đầu ra là Bằng chứng suy dẫn. Truyền ID cha của nó và gói đã lưu vào `inspect-analysis-view --evidence-bundle module.evidence.json` cho các truy vấn ngoại tuyến. Các view MCP đọc Bằng chứng được lưu giữ và các view ngoại tuyến đọc gói; cả hai đều không yêu cầu tệp pyc gốc phải còn tồn tại. Ngân sách phân tích cú pháp và việc lưu giữ toàn bộ nguồn vẫn áp dụng cho phân phối tóm tắt.

## Quan sát (Observations)

Kết quả Bằng chứng (Evidence) nội tuyến chứa đường dẫn hiện vật, SHA-256 và độ dài byte. `header` chứa:

- `magic`, trường 16-bit little-endian;
- `python_version` và `code_layout`, được ánh xạ từ bảng cố định CPython 3.7–3.13 (`py37`, `py38`, hoặc `py311`); magic 3.11 alpha trước 3494 sử dụng các bố cục mã khác và không được hỗ trợ. Magic 3.7.0a1/a2 3390/3391 sử dụng header 12 byte và nằm ngoài định dạng được hỗ trợ của bộ đọc này;
- `flags`, `hash_based`, và `check_source` từ PEP 552;
- `source_hash_hex` cho các tệp pyc dựa trên hash, hoặc `source_mtime` và `source_size` cho các tệp pyc dựa trên timestamp.

`code_objects` liệt kê từng đối tượng mã có thể tiếp cận được trong đồ thị marshal đã giải mã một lần, mã mô-đun cấp cao nhất trước tiên. Điều này tuân theo hành vi vùng chứa của CPython; nó không phải là một kho kiểm kê các bản ghi đối tượng mã được tuần tự hóa thô. `nested_code` chứa các chỉ mục của các đối tượng mã riêng biệt có thể tiếp cận được từ các hằng số của đối tượng đó, bao gồm mã lồng trong tuple và các vùng chứa khác và chính chủ sở hữu khi một vùng chứa hằng số tiếp cận nó.
Một hàm được chia sẻ thông qua tham chiếu marshal được đặt tên theo chỉ mục một lần cho mỗi chủ sở hữu, ngay cả khi chủ sở hữu đó tiếp cận lại nó thông qua cùng một vùng chứa. Các mối quan hệ được thu thập sau khi đồ thị marshal hoàn tất, do đó một vùng chứa mà một đối tượng mã lồng nhau cũng tham chiếu đến sẽ không được tóm tắt trong khi các mục của nó vẫn còn thiếu. Mỗi đối tượng báo cáo tên, qualname tùy chọn, tên tệp, giá trị `co_firstlineno` 32-bit có dấu, số lượng đối số, cờ, tên biến, biến cục bộ (hoặc `localsplusnames` trên 3.11+), tóm tắt hằng số (const summaries), và độ dài bytecode cộng với SHA-256.

Sự bằng nhau của đối tượng mã được sử dụng bởi các tập hợp và từ điển tuân theo định dạng CPython đã chọn: 3.7–3.10 suy dẫn `CO_NOFREE` từ bảng free/cell để so sánh bằng nhau, trong khi 3.11+ so sánh các cờ thô; `co_flags` thô vẫn nằm trong các dữ kiện được báo cáo. Các bố cục cũ hơn bỏ qua bảng dòng, trong khi 3.11+ bao gồm bảng dòng và ngoại lệ và bỏ qua các đơn vị mã inline-cache theo bảng cache cho magic chính xác. Độ dài bytecode và SHA-256 được báo cáo vẫn mô tả các byte mã ban đầu. Điều này không mô phỏng quá trình deoptimization của CPython đối với các byte opcode chuyên biệt hoặc được can thiệp (instrumented) trong các tệp pyc được tạo thủ công.
Mã băm SHA-256 của hiện vật xác định chính xác các byte `.pyc` đã chọn độc lập với đồ thị đối tượng đã giải mã này. Các vị trí tham chiếu tuân theo hành vi CPython của từng thẻ: các giá trị singleton và `TYPE_REF` không bảo lưu các vị trí mới ngay cả khi thẻ của chúng có đặt `FLAG_REF`.

Tên được báo cáo là chuỗi hiển thị JavaScript; các tên Unicode CPython riêng biệt do đó có thể có cùng giá trị hiển thị và không phải lúc nào cũng có thể được tái tạo từ trường đó một mình. Tính bằng nhau của mã sử dụng các byte chuỗi marshal được lưu giữ, trong khi SHA-256 của hiện vật tiếp tục xác định các byte tệp gốc.

Bytecode không được dịch ngược. Các chuỗi hằng số có thể bị cắt bớt trong bản tóm tắt trong khi lần duyệt marshal vẫn tiêu thụ các byte ban đầu; `bytes` của tóm tắt chuỗi là độ dài marshal đó. Hằng số số nguyên báo cáo `bits`; `value` thập phân của chúng là `null` trên 256 bit. Các phần thực và ảo không hữu hạn sử dụng cách viết `inf`, `-inf`, và `nan` của CPython; số 0 âm sử dụng `"-0.0"` để JSON bảo toàn dấu của nó. Bản xem trước chuỗi bảo toàn các surrogate đơn lẻ nguồn nhưng không bao giờ chia tách một cặp surrogate hợp lệ tại ranh giới xem trước. Độ dài từ điển và tập hợp tuân theo CPython: các khóa và thành viên không thể băm sẽ làm cho lần duyệt thất bại, các thành viên tập hợp bằng nhau chỉ tính một lần, các khóa từ điển bằng nhau giữ lại đối tượng khóa đầu tiên và chỉ thay thế giá trị của nó, các đối tượng mã là các khóa và thành viên hợp lệ chỉ khi tất cả hằng số của chúng có thể băm được, và một giá trị từ điển null sẽ kết thúc dict mà không có mục đó.

## Giới hạn và lỗi (Limits & Failures)

Tệp được chọn được giới hạn ở 16 MiB theo mặc định (tối đa 64 MiB). Số lượng đối tượng marshal, độ sâu lồng nhau và kích thước mỗi chuỗi đều có giới hạn; vượt quá ngân sách sẽ thất bại mà không có thành công một phần. Các lỗi giới hạn giải mã mang tính xác định: hành động khôi phục nêu tên trường đã chọn và tùy chọn CLI, giá trị hiệu dụng của nó và mức tối đa được chấp nhận. Hãy nâng trường đó trong lệnh gọi tiếp theo khi mức tối đa cho phép; việc đợi phân tích khác hoặc tăng bộ nhớ máy chủ không làm tăng các quyền kiểm soát này. Số lượng đối số, số lượng chỉ theo vị trí, số lượng chỉ theo từ khóa và số lượng biến cục bộ phải không âm. `co_stacksize` và `co_flags` được giữ lại dưới dạng các trường nguồn 32-bit có dấu khi hàm tạo CPython đã chọn chấp nhận chúng. Hàm tạo CPython 3.7 và 3.8 alpha qua 3.8.0a4 không từ chối kích thước stack hoặc cờ âm; việc kiểm tra đã được thêm vào bởi [commit CPython a9f05d69](https://github.com/python/cpython/commit/a9f05d69ccbf3c75cdd604c25094282697789a62).
Magic 3410 là mơ hồ: cả 3.8.0a4 và 3.8.0b1 đều sử dụng nó, nhưng chúng nằm ở hai phía đối diện của thay đổi hàm tạo đó. Các giá trị âm dưới magic 3410 được bảo toàn với một giới hạn rõ ràng vì header không thể xác định bản sửa đổi hàm tạo nào đã tạo ra chúng. Magic 3411 và các magic 3.8 sau đó được giới thiệu sau thay đổi đó, do đó hàm tạo của chúng từ chối các giá trị âm. Các mục magic cố định phân biệt những trường hợp này; xem [hàm tạo CPython 3.7.0](https://github.com/python/cpython/blob/v3.7.0/Objects/codeobject.c), [hàm tạo 3.8.0a4](https://github.com/python/cpython/blob/v3.8.0a4/Objects/codeobject.c), [hàm tạo 3.8.0b1](https://github.com/python/cpython/blob/v3.8.0b1/Objects/codeobject.c), và [bảng magic 3.8.0b2](https://github.com/python/cpython/blob/v3.8.0b2/Lib/importlib/_bootstrap_external.py).

Độ bao phủ marshal hoàn chỉnh có nghĩa là tất cả các trường trong tệp đã chọn đều được giải mã; nó không xác lập rằng CPython sẽ chấp nhận hoặc thực thi đối tượng mã đó. Trên bố cục 3.11+, `co_code` phải bao phủ một số nguyên các đơn vị mã. Mọi bố cục đều từ chối số lượng đối số không khớp với các kiểm tra CPython của bảng cục bộ (`co_varnames`, bao gồm `*args` / `**kwargs`, hoặc fast locals trên 3.11+). Các tệp đó là sai định dạng, do đó quy trình làm việc không báo cáo độ bao phủ hoàn chỉnh. 3.7–3.10 vẫn chấp nhận `co_code` lẻ, khớp với các hàm tạo đó. Lần duyệt thu thập mã lồng nhau lần theo các chuỗi tham chiếu một cách lặp đi lặp lại và giữ một danh sách hậu duệ dùng chung cho một vùng chứa chỉ đơn thuần bọc vùng chứa khác, thay vì sao chép danh sách đó vào mọi wrapper. Các khóa tập hợp và từ điển lần theo chuỗi tham chiếu tuple và frozenset theo cùng một cách, do đó một chuỗi `TYPE_REF` dài vẫn nằm trong call stack. Một phần tử con dùng chung được lưu trữ hai lần vẫn tính khóa mở rộng của nó vào ngân sách chuỗi. Khóa bằng nhau của đối tượng mã ghi lại độ dài của từng bảng tên và ranh giới của từng mục, do đó `()` và `("",)` vẫn riêng biệt, và `co_varnames`, `co_freevars`, và `co_cellvars` vẫn là các bảng riêng biệt. Tên lặp lại thông qua tham chiếu marshal, bao gồm từng tên hằng số mã mà báo cáo sao chép, tính vào ngân sách tệp một lần cho mỗi bản sao. Chuỗi rỗng không thêm byte nào, do đó mọi mục bảng tên được sao chép (`co_names`, varnames, freevars, cellvars, và `localsplusnames`) đều tính vào ngân sách đối tượng trước khi mảng của nó được xây dựng. Các bảng hằng số được sao chép và từng mối quan hệ mã lồng nhau được báo cáo tính vào cùng một ngân sách đối tượng đó. Các chữ số nguyên và khóa tập hợp hoặc từ điển mở rộng, bao gồm một tuple dùng chung lặp lại ở mỗi cấp độ, tính vào ngân sách chuỗi. Magic nằm ngoài bảng 3.7–3.13 cố định, header bị cắt cụt, byte marshal phía sau và đối tượng cấp cao nhất không phải mã là các lỗi có kiểu.

Một header đã giải mã không chứng minh tệp `.py` khớp có mặt. `python_version` là bảng magic, không phải một trình thông dịch đang chạy. Quy trình làm việc này không khôi phục mã nguồn.
