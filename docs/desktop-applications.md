# Thư mục cài đặt ứng dụng Desktop (Desktop Installation Directories)

Sử dụng `inspect_desktop_application` khi mục tiêu là một thư mục cài đặt và runtime của nó chưa xác định. Công cụ này đọc các mục trong thư mục, các header được chọn và metadata có giới hạn, sau đó báo cáo các ứng viên runtime, tệp hỗ trợ, dấu hiệu đường dẫn thành phần và các thao tác theo dõi cụ thể. Không cần provider native hoặc phiên nhị phân đang hoạt động. Công cụ không chạy ứng dụng, không giải nén kho lưu trữ, không lần theo liên kết tượng trưng con (child symlinks), không liên hệ với các dịch vụ và không làm thay đổi các tệp mục tiêu.

## MCP

```json
{ "path": "/artifacts/MyApplication" }
```

Mặc định `detail: "summary"` lưu giữ toàn bộ Bằng chứng (Evidence) nguồn trong phiên làm việc và trả về các ứng viên cùng đường dẫn hỗ trợ tiêu biểu, số lượng, độ bao phủ, giới hạn và nhóm các đầu dò tiếp theo. `normalized_result.parent_evidence_id` xác định bản ghi hoàn chỉnh. Server không có tính năng lưu giữ Bằng chứng sẽ từ chối phân phối tóm tắt trước khi đọc mục tiêu; hãy chọn `detail: "complete"` để nhận nội tuyến mọi dữ kiện được lưu giữ.

Đọc chính xác các tệp trong một yêu cầu:

```json
{
  "source": {
    "kind": "retained-evidence",
    "evidence_id": "ev_<parent-sha256>"
  },
  "view": {
    "kind": "desktop-application",
    "collection": "files",
    "paths": ["Game_Data/Managed/Assembly-CSharp.dll", "GameAssembly.dll"],
    "fields": ["path", "read_status", "ranges", "facts", "issues"],
    "offset": 0,
    "limit": 16
  }
}
```

Các bộ sưu tập gồm `files`, `omissions`, `candidates`, `components` và `next_probes`.
Tệp và mục bỏ sót (omissions) chấp nhận `paths` chính xác và các đợt `file_ids` / `omission_ids`.
Tệp cũng chấp nhận `has_issues: true` để chọn trực tiếp chẩn đoán đầu dò, hoặc `false` để chọn các hàng không có sự cố được báo cáo. Bộ lọc này sử dụng các dữ kiện sự cố được lưu giữ; nó không đọc lại mục tiêu hoặc suy diễn rằng các tệp chưa được kiểm tra là bình thường. Khi cả hai được cung cấp, tất cả bộ lọc sẽ được áp dụng; việc chọn trống không trả về hàng nào. Bản ghi bỏ sót lưu giữ đường dẫn quan sát được, loại và `omission_id` ổn định; payload của chúng không được đọc và metadata mục đặc biệt khác vẫn không khả dụng.
`candidates` chấp nhận các `families` chính xác.
Bộ chọn bị bỏ qua sẽ khớp với tất cả các hàng được lưu giữ; một bộ chọn trống không khớp với hàng nào.
Mọi bộ sưu tập đều hỗ trợ `fields`, `offset` và `limit`; ID hàng ổn định của nó luôn được bao gồm. Việc chọn chỉ thay đổi phân phối, không thay đổi kiểm tra nguồn. Các trang giữ nguyên độ bao phủ nguồn và liên kết Bằng chứng cha. Hết trang không chứng minh sự vắng mặt trong một nguồn một phần. Bằng chứng hoàn chỉnh vẫn có thể truy cập được thông qua `get_evidence_bundle` và xuất gói Bằng chứng.

`probe_issues` của bản tóm tắt đếm các tệp có sự cố được báo cáo và các mục sự cố được lưu giữ của chúng. Nó xem trước tối đa hai tệp như vậy, bao gồm `file_id` ổn định, trạng thái đọc và sự cố đầu tiên. Xem trước đường dẫn/lý do sử dụng tối đa 256 đơn vị UTF-16 mỗi đơn vị mà không chia tách các cặp surrogate; `text_truncated` đánh dấu văn bản được rút gọn và `preview_truncated` đánh dấu các tệp bổ sung. Đây là các tiền tố hiển thị; toàn bộ đường dẫn của bên gọi và tất cả các lý do vẫn còn trong Bằng chứng. Số lượng mô tả các quan sát được lưu giữ, trong khi độ bao phủ nguồn vẫn xác định việc kiểm tra một phần.

Sử dụng `inspect_analysis_view` với `collection: "files", has_issues: true` và `fields: ["path", "read_status", "issues"]` để lấy các lý do chính xác, với việc phân trang và liên kết cha như thông thường. Nó giao cắt với các đường dẫn chính xác và ID tệp theo lô.

Đầu dò kiểm kê ASAR trước tiên chọn đường dẫn chính xác của nó bằng `open_binary`; sau đó gọi `inspect_artifact` trên mục tiêu đang hoạt động đó. Bản thân việc kiểm tra thư mục không chọn hoặc thay thế mục tiêu của phiên làm việc. Quy trình làm việc của ứng dụng JavaScript chấp nhận trực tiếp đường dẫn đã chọn của riêng nó.

## CLI

CLI một lần chạy (one-shot CLI) trả về Bằng chứng hoàn chỉnh theo mặc định. Đối với đầu ra nhỏ gọn, hãy lưu giữ rõ ràng nó trong một gói:

```bash
rea inspect-desktop-application /artifacts/MyApplication \
  --detail summary --evidence_output /reports/application.evidence.json --json
```

Lệnh này trả về Bằng chứng `summary` và biên nhận xuất bản `evidence_bundle`. Các tệp đầu ra hiện có không bao giờ bị ghi đè. Phân phối tóm tắt yêu cầu `--evidence_output`, được kiểm tra trước khi phân tích. `inspect-evidence-bundle` chọn các bản ghi đã lưu; một bản ghi đã lưu chính xác cũng có thể cung cấp dữ liệu cho `inspect-analysis-view`. `--detail complete` trả về Bằng chứng nguồn nội tuyến và có thể tùy chọn ghi cùng một gói. Thao tác xuất chỉ ghi tệp đầu ra do bên gọi chọn.

## Dữ kiện và tính không chắc chắn (Facts and Uncertainty)

| Nhóm (Family) | Bằng chứng được sử dụng |
| :--- | :--- |
| .NET | Header tùy chọn PE khai báo một thư mục CLI không rỗng. |
| Unity Mono | Một assembly được quản lý bên dưới `*_Data/Managed` hoặc `Contents/Resources/Data/Managed` có tham chiếu assembly UnityEngine chính xác. |
| Unity IL2CPP | Magic metadata IL2CPP được nhận diện (kèm phiên bản thô khi hoàn chỉnh) cùng tồn tại với tệp ảnh native được đặt tên theo quy ước `GameAssembly.dll`, `GameAssembly.so`, `GameAssembly.dylib` hoặc `libil2cpp.so`. Sự tương ứng của bản dựng vẫn chưa được biết. |
| Godot | Header `GDPC` ở đầu tệp hoặc độ lệch đuôi tệp thực thi (executable trailer offset) đã được xác thực. |
| GameMaker | Một vùng chứa `FORM` có giới hạn bắt đầu bằng đoạn `GEN8`. Quá trình quét dò tìm các tệp dữ liệu quy ước `data.win`, `game.unx`, và `game.ios` để tìm header đó. |
| Unreal | Quy ước đường dẫn `Content/Paks` hoặc chữ ký IoStore TOC; tên tệp `.pak` tùy ý là không đủ. |
| Electron | Quy ước đường dẫn `resources/app.asar` hoặc `resources/default_app.asar`; tính hợp lệ của kho lưu trữ chưa được xác minh. |

Khi magic metadata 4 byte có mặt nhưng các byte phiên bản chưa hoàn chỉnh, dữ kiện lưu giữ `raw_version: null`; nó không đưa ra đầu dò kiểm tra metadata IL2CPP. Tiền tố magic/phiên bản hoàn chỉnh với phiên bản thô được hỗ trợ 24, 27, 29, 31 hoặc 39 cung cấp một bước theo dõi `inspect_il2cpp_metadata` tại đường dẫn tệp được tiếp nhận của nó, bao gồm cả khi không tìm thấy thư viện native. Đầu dò này không xác lập tính hợp lệ đầy đủ của metadata hoặc sự tương ứng với tệp nhị phân native; việc kiểm tra phiên bản 24 vẫn chỉ áp dụng cho các bố cục header được hỗ trợ. Xem [metadata IL2CPP](il2cpp-metadata.md).

Các ứng viên được rút ra hoặc suy luận từ những dữ kiện này và không chứng minh việc tải hoặc thực thi. Các nhãn BepInEx, MelonLoader, Easy Anti-Cheat và BattlEye chỉ xác định quy ước đường dẫn; sự hiện diện khi chạy vẫn chưa được biết. Không cung cấp cơ chế bypass nào.

`file_id` bao gồm namespace gốc được tiếp nhận, đường dẫn được ghi lại và danh tính hệ thống tệp khả dụng. Đường dẫn gốc do bên gọi chọn và danh tính gốc/đường dẫn chuẩn được tiếp nhận đều được lưu giữ; `local_path` xác định từng tệp được quan sát cho các thao tác tiếp theo. Gốc được chọn có thể là liên kết tượng trưng; các liên kết tượng trưng con không được lần theo. Số nguyên metadata chính xác vẫn là chuỗi thập phân. Các phạm vi đọc có độ lệch, số lượng byte và mã băm SHA-256; dữ kiện có kiểu để lộ RVA của CLI, phiên bản metadata/gói và mã định danh assembly (assembly tokens). Danh tính thư mục và liên kết tượng trưng không khả dụng từ quá trình duyệt vẫn là null.

`snapshot_sha256` của thư mục cam kết các đường dẫn được lưu giữ, các loại bỏ sót, metadata khả dụng và mã băm phạm vi, được hợp thành thông qua danh tính gốc, mục và điểm bỏ sót được tiếp nhận. Trần mức bên gọi, bộ đếm độ bao phủ, cách diễn đạt chẩn đoán và dữ kiện giải thích vẫn nằm trong Bằng chứng, bên ngoài danh tính nguồn. Các quan sát giống nhau dưới các giới hạn hào phóng khác nhau chia sẻ cùng một mã băm snapshot; ID Bằng chứng vẫn bảo toàn các tham số yêu cầu và độ bao phủ được báo cáo. Các mục được lưu giữ hoặc phạm vi đọc khác nhau sẽ làm thay đổi snapshot nguồn.
Bản ghi bỏ sót chia sẻ ngân sách mục và biểu diễn và có thể được tìm nạp độc lập thông qua chế độ xem của chúng.
**Đây không phải là mã băm của tất cả nội dung tệp.** Đây là các quan sát mục ổn định tuần tự, không phải là một snapshot thư mục nguyên tử. Các byte chưa đọc vẫn là chưa xác định.

## Giới hạn tài nguyên và tái sử dụng (Resource Bounds & Reuse)

Mặc định là 20.000 mục được kiểm tra và tổng cộng 8 MiB lượt đọc đầu dò. Bên gọi có thể chọn tối đa 100.000 mục và 64 MiB. Ngân sách biểu diễn 32 MiB riêng biệt giới hạn các hàng, đường dẫn và dữ kiện được lưu giữ trước khi tăng trưởng. Mỗi đường dẫn được lưu giữ được tính theo mức lớn hơn giữa kích thước heap UTF-16 và kích thước JSON đã escape của nó, do đó các đường dẫn gói macOS điển hình chứa được khoảng 20.000 hàng, và các đường dẫn có ký tự điều khiển thì ít hơn. Giới hạn và đầu dò thất bại tạo ra độ bao phủ một phần. Duyệt và đọc là tuần tự. Mỗi bộ đọc tiếp nhận một cuộc kiểm tra tại một thời điểm; các yêu cầu chồng chéo sẽ nhận được sự từ chối dung lượng có thể thử lại mà không hủy bỏ bên gọi đã được tiếp nhận.

Việc giải mã tham chiếu Unity dừng lại sau một nhân chứng (witness) trong mỗi thư mục `Data/Managed`; tham chiếu của các assembly khác vẫn chưa được kiểm tra, trong khi header của chúng vẫn được kiểm tra. Dữ kiện AssemblyRef chỉ giải mã tên và mã định danh; chúng không giải mã hoặc băm các payload khóa/văn hóa/mã băm không liên quan hoặc xác lập liên kết runtime. Nội dung gói (pack contents), dữ liệu được mã hóa, sự tương ứng metadata/địa chỉ và PCK nhúng section không có phần đuôi là các năng lực riêng biệt.

Kết quả hệ thống tệp không được lưu trong bộ nhớ đệm qua các lệnh gọi. Chế độ xem được lưu giữ tái sử dụng metadata bất biến đã được xác thực mà không cần quét lại tệp hoặc lưu giữ thêm một kho kiểm kê khác.
