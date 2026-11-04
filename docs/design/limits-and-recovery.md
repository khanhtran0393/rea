# Cài đặt phân tích, giới hạn và phục hồi (Analysis settings, limits, and recovery)

Hướng dẫn này giải thích thiết kế cài đặt thời gian chạy (runtime settings) cho người bảo trì và nhà phát triển: ai sở hữu từng giá trị, khi nào bản cập nhật có hiệu lực, cách các provider thực thi nó và cách mở rộng hệ thống. Hợp đồng công cụ công khai nằm trong [Hợp đồng MCP](../mcp-contracts.md#editable-analysis-settings); các kiểm tra thời gian chạy được ghi nhận và giới hạn của chúng nằm trong [báo cáo xác minh](../verification/runtime-analysis-settings.md).

## Vấn đề và ranh giới thiết kế (Problem and design boundary)

Một khối lượng công việc hợp lệ có thể vượt quá giá trị mặc định thận trọng. Tiêu đề ASAR quá khổ có thể ngăn chặn phân tích toàn bộ kho lưu trữ; một hàm Ghidra chậm có thể vượt quá thời hạn dịch ngược của nó. Việc yêu cầu một tiến trình REA mới để thay đổi ngân sách sẽ làm gián đoạn cuộc điều tra và có thể loại bỏ trạng thái provider có thể tái sử dụng.

REA cung cấp một tập hợp các giá trị mặc định phân tích được tuyển chọn thông qua MCP để bên gọi có thể điều chỉnh giới hạn được báo cáo và thử lại cùng một khối lượng công việc trong runtime đang chạy. Mỗi cài đặt có một giá trị được định kiểu, ranh giới tài nguyên, chủ thể thực thi và điểm áp dụng xác định. Thiết kế này áp dụng trên toàn bộ quá trình tiếp nhận kho lưu trữ, khởi động provider, thời hạn hoạt động và yêu cầu bộ nhớ của worker.

Hàm setter thay đổi các giá trị mặc định trong bộ nhớ. Nó không chỉnh sửa các biến môi trường tùy ý, tệp cấu hình, đăng ký MCP hoặc cài đặt provider. Các cơ chế đó có các chủ thể sở hữu và vòng đời khác nhau. Nó cũng không thể thay đổi thời hạn của client, mở rộng công việc đã chấp nhận, giải phóng tài nguyên được lưu giữ hoặc xóa giới hạn biểu diễn cố định. Việc cung cấp một cài đặt đòi hỏi một đường dẫn thực thi hoạt động thực tế, không chỉ là một mục trong lược đồ công cụ.

Các mức trần tiếp theo nên trở thành cài đặt được theo dõi trong [#6176](https://github.com/morluto/rea/issues/6176). Tính trung thực trong việc đặt tên cho các lỗi giới hạn được theo dõi trong [#5910](https://github.com/morluto/rea/issues/5910).

## Quyền sở hữu và luồng dữ liệu (Ownership and data flow)

| Chủ thể sở hữu | Trách nhiệm |
| :--- | :--- |
| [Định nghĩa miền](https://github.com/morluto/rea/blob/main/src/domain/analysisSettingDefinitions.ts) và [kiểu](https://github.com/morluto/rea/blob/main/src/domain/analysisSettings.ts) | ID cài đặt chính xác, kiểu giá trị, ranh giới/mặc định dùng chung, đơn vị, liên kết provider/thao tác, điểm áp dụng và metadata giới hạn có cấu trúc. Hằng số số học ở lại với chủ sở hữu miền của chúng. |
| [Bộ phân tích cú pháp khởi chạy](https://github.com/morluto/rea/blob/main/src/config/analysisSettings.ts) | Phân tích cú pháp các giá trị khởi chạy đã chọn một lần, bảo toàn giá trị không hợp lệ dưới dạng lỗi cấu hình có định kiểu thay vì thay thế bằng giá trị mặc định. |
| [AnalysisSettings](https://github.com/morluto/rea/blob/main/src/application/AnalysisSettings.ts) | Sở hữu mốc cơ sở khởi chạy của một runtime, các bản sửa đổi ghi đè bất biến, cập nhật nguyên tử, kiểm tra và bản chụp thao tác được chấp nhận. Nó không sở hữu vòng đời hoặc việc dọn dẹp của provider. |
| [Thành phần](https://github.com/morluto/rea/blob/main/src/main/eagerRuntime.ts) và [chế độ xem cấu hình có kiểu](https://github.com/morluto/rea/blob/main/src/config/runtimeAnalysisSettings.ts) | Tiêm cùng một reader cài đặt vào các provider và quy trình làm việc; dịch các ghi đè thành cấu hình mà mỗi consumer hiểu được. |
| Chủ sở hữu provider hoặc tài nguyên | Giải quyết ngân sách hiệu quả, thực thi nó tại ranh giới áp dụng và gắn giá trị thực tế cùng nguồn đã biết vào các lỗi. |
| [Ranh giới đăng ký MCP](https://github.com/morluto/rea/blob/main/src/server/EvidenceMcpServerCore.ts) | Đi vào phạm vi cài đặt của thao tác trước khi gọi trình xử lý của nó. |
| [Hợp đồng cài đặt](https://github.com/morluto/rea/blob/main/src/contracts/analysisSettingsContracts.ts) và [đăng ký công cụ](https://github.com/morluto/rea/blob/main/src/server/registerAnalysisSettingsTools.ts) | Xác thực các yêu cầu có tên và cung cấp kết quả kiểm tra/cập nhật mà không yêu cầu mục tiêu đang mở hoặc provider phân tích khỏe mạnh. |

Đường dẫn thông thường là: thành phần tạo kho lưu trữ, trình xử lý MCP chụp lại bản sửa đổi hiện tại của nó và các provider đọc bản chụp đó khi giải quyết ngân sách. Nếu một chủ sở hữu từ chối công việc, nó sẽ báo cáo giới hạn hiệu quả thông qua phép chiếu lỗi dùng chung. Một bản cập nhật cài đặt tiếp theo sẽ xuất bản một bản sửa đổi mới cho các trình xử lý sau này. Nó không can thiệp vào thao tác đã thất bại hoặc tự động thử lại.

### Bản chụp thao tác được chấp nhận (Accepted-operation snapshots)

`AnalysisSettings.run()` cài đặt bản chụp bất biến hiện tại trong `AsyncLocalStorage`. Hàm `read` của nó trả về bản chụp đó bên trong thao tác và bản sửa đổi mới nhất bên ngoài thao tác. Các provider phụ thuộc vào hợp đồng reader thay vì kho lưu trữ có thể thay đổi hoặc việc tra cứu môi trường xung quanh.

Sự chấp nhận ở đây là việc đi vào phạm vi cài đặt của trình xử lý đã đăng ký, không phải việc các byte đến phương tiện truyền tải MCP. Công việc được xếp hàng hoặc chờ đợi trong phạm vi đó sẽ giữ nguyên bản chụp của nó. Việc khởi tạo máy chủ bị trì hoãn vẫn phải điều phối thông qua trình xử lý có phạm vi chuẩn.

| Sự kiện | Bản sửa đổi được sử dụng |
| :--- | :--- |
| Thao tác A đi vào trình xử lý của nó, sau đó chờ một provider | Bản sửa đổi hiện tại 0 |
| Cập nhật cài đặt xuất bản bản sửa đổi 1 | Các trình xử lý trong tương lai có thể đọc bản sửa đổi 1 |
| Thao tác A tiếp tục và bắt đầu worker của nó | Bản sửa đổi được chấp nhận của nó 0 |
| Thao tác B đi vào trình xử lý của nó | Bản sửa đổi 1 |

Điều này mang lại cho mỗi thao tác ngữ nghĩa ngân sách ổn định ngay cả khi một kết nối khác cập nhật cài đặt trong khi nó đang chờ. `next_worker` và `next_startup` mô tả nơi một giá trị được chấp nhận được áp dụng; chúng không làm cho các worker hoặc lượt khởi động thuộc về một thao tác cũ hơn chấp nhận bản sửa đổi mới hơn.

Kiểm tra chủ ý đọc các giá trị mặc định hiện tại của kho lưu trữ, thay vì bản chụp của thao tác gọi. Nó báo cáo `active_values.status: "not_observed"`: kết quả kiểm tra không thể xác lập ngân sách hoặc thời gian còn lại của một worker đang chạy.

### Thành phần và nhúng (Composition and embedding)

Runtime MCP tạo một kho lưu trữ và cung cấp reader của nó cho các nhà máy provider. `createServer` sử dụng kho lưu trữ được cung cấp hoặc tạo một kho lưu trữ, và kết nối các provider mặc định của nó với kho lưu trữ đó. Một bên nhúng cung cấp phiên nhị phân tùy chỉnh hoặc provider khác cũng phải tiêm cùng một reader vào thành phần của provider đó và truyền kho lưu trữ tới `createServer`. Chỉ riêng việc cung cấp các công cụ cài đặt không thể làm cho một provider tùy chỉnh tiếp nhận các bản cập nhật.

Giữ các nhà máy mặc định, provider được cung cấp rõ ràng, thành phần truyền tải và điều phối trình xử lý bị trì hoãn được bao phủ riêng biệt khi hệ thống dây nối của chúng khác nhau. Giải quyết các cài đặt tại ranh giới thao tác; sao chép giá trị runtime vào hàm tạo provider sẽ làm đóng băng giá trị đó khi khởi chạy. Đối với các điểm vào bên ngoài wrapper MCP, một bên nhúng cần sự đảm bảo bản chụp tương tự phải thiết lập phạm vi thao tác với `AnalysisSettings.run()`.

## Đặt tên giá trị mặc định, sau đó là cài đặt khi người vận hành cần thêm khoảng trống

Đặt con số trong một hằng số có tên kèm theo đơn vị. Không để nó dưới dạng một con số ma thuật (magic number) tại nơi gọi.

Khi một khối lượng công việc có thể cần thêm dung lượng, hãy thêm một cài đặt có giá trị chưa đặt là hằng số đó. Các hợp đồng số học thuần túy nằm trong `src/domain/analysisResourceLimits.ts` hoặc chủ sở hữu miền hiện có của chúng. `analysisSettingDefinitions.ts` liên kết các ID runtime được tuyển chọn với các ranh giới, mặc định và điểm áp dụng tương tự. Trình phân tích cú pháp môi trường và văn bản phục hồi ở lại dưới `src/config/`; đăng ký lược đồ khởi chạy trên `environmentSchema` để quá trình khởi động và doctor thống nhất. Thêm cài đặt khởi chạy vào `SELECTED_REGISTRATION_SETTINGS` khi thiết lập cần chuyển tiếp chúng. Chưa đặt sẽ giữ nguyên hành vi hiện tại. Một giá trị khởi chạy không hợp lệ vẫn là một lỗi cấu hình, không bao giờ tự động chuyển về mặc định; ranh giới xác thực của nó xác định liệu MCP có thể khởi động hay không (xem [cấu hình khởi chạy không hợp lệ](#cấu-hình-khởi-chạy-không-hợp-lệ)). Một lệnh ghi runtime không hợp lệ là `invalid_request` và không thay đổi gì cả.

Ví dụ: `regex.matching_deadline_ms` và hạt giống khởi chạy `REA_REGEX_MATCHING_DEADLINE_MS` của nó chia sẻ mặc định 5000 ms, mức sàn 1 ms và mức tối đa 2147483647 ms. Node lên lịch độ trễ `setTimeout` dài hơn dưới dạng 1 ms. Việc thay đổi cài đặt thời gian chạy ảnh hưởng đến tìm kiếm được chấp nhận tiếp theo; thay đổi môi trường khởi chạy ảnh hưởng đến tiến trình REA tiếp theo.

[`REA_MCP_MAX_RESPONSE_BYTES`](https://github.com/morluto/rea/blob/main/src/config/mcpResponseBudget.ts) là hình dạng ngân sách byte: một số nguyên an toàn thập phân với mức sàn 10 MiB (`MINIMUM_RESPONSE_BYTES`), cùng lược đồ dùng chung và không có giá trị dự phòng ngầm. Thay đổi cần khởi động lại REA và bộ đệm nhận client phù hợp. Nó không làm tăng giới hạn chuỗi dành riêng của REA hoặc trần chuỗi của Node. Danh sách cho phép thiết lập không sao chép cài đặt này. Một cài đặt khoảng trống mới nên làm theo thời hạn regex.

Ngân sách cho mỗi lệnh gọi có thể có mặc định thời gian chạy mà không cần có cài đặt môi trường. JavaScript `max_heap_mb` (CLI `--max-heap-mb`) là một số nguyên từ 128 đến 16384 MiB, mặc định 1024. Nó áp dụng trên phân tích tiếp theo, bắt đầu một worker mới và không khởi động lại REA. Nó không làm tăng ngân sách kết quả cha, trần truyền tải hoặc giới hạn trình phân tích cú pháp cố định.

Bộ nhớ máy chủ và `NODE_OPTIONS` không nâng cao mức tiếp nhận của REA trừ khi sự phục hồi của mức tiếp nhận đó nêu rõ như vậy. Ngân sách kết quả cha của JavaScript là một trường hợp như vậy: sự phục hồi của nó yêu cầu thiết lập `NODE_OPTIONS` (ví dụ: `--max-old-space-size=<MiB>`) trên tiến trình bắt đầu REA, sau đó khởi động lại tiến trình đó. `max_heap_mb` không làm tăng nó.

## Cài đặt thời gian chạy thông qua MCP (Runtime settings through MCP)

`inspect_analysis_settings` liệt kê các cài đặt được tuyển chọn, giá trị hiện tại và khởi chạy của chúng, nguồn, ranh giới, điểm áp dụng và tên thao tác liên quan. Bộ lọc theo ID cài đặt chính xác, provider hoặc thao tác giao nhau. Các giá trị worker đang hoạt động rõ ràng là `not_observed`; một giá trị mặc định được cấu hình không phải là phép đo công việc đang hoạt động. Kiểm tra cũng tách biệt các giới hạn máy chủ đã biết khỏi các thời hạn client và bộ đệm nhận chưa biết.

`update_analysis_settings` chấp nhận một mảng `changes` nguyên tử. Mỗi mục đặt tên một `setting` chính xác và `action: "set"` với `value` đã được xác thực lược đồ, hoặc `action: "reset"`. Reset khôi phục giá trị khởi chạy. `expected_revision` là một bộ bảo vệ so sánh và đặt tùy chọn; xung đột và khóa trùng lặp không thay đổi điều gì. Một lệnh ghi không thay đổi sẽ không làm tăng bản sửa đổi.

Bản sửa đổi theo dõi trạng thái ghi đè, không chỉ các giá trị số hiệu quả. Đặt ghi đè rõ ràng bằng giá trị khởi chạy vẫn thay đổi nguồn của nó thành `runtime` và nâng cao bản sửa đổi. Reset sẽ xóa phần ghi đè đó, ngay cả khi số hiệu quả vẫn giữ nguyên. Xác thực, kiểm tra sửa đổi và kiểm tra khóa trùng lặp đều diễn ra trước khi xuất bản bản chụp thay thế; một lô bị từ chối không thể áp dụng từng phần. Các client điều phối ghi có thể cung cấp `expected_revision`; nếu không có nó, các bản cập nhật hợp lệ sẽ áp dụng cho trạng thái hiện tại.

| Cài đặt thời gian chạy | Điểm áp dụng |
| :--- | :--- |
| `asar.max_header_bytes` | Thao tác lưu trữ tiếp theo, bao gồm phân tích toàn bộ ứng dụng JavaScript |
| `ghidra.startup_timeout_ms`, `hopper.startup_timeout_ms` | Nỗ lực khởi động provider tiếp theo |
| `ghidra.decompile_timeout_seconds` | Thao tác tiếp theo trên Program Ghidra hiện có; `null` có nghĩa là không giới hạn |
| `jadx.operation_timeout_ms` | Thao tác JADX tiếp theo |
| `regex.matching_deadline_ms`, `regex.worker_heap_request_mib` | Tìm kiếm tiếp theo / worker regex mới |
| `capture.decoder_old_generation_mib` | Bộ giải mã capture thuộc sở hữu tiếp theo |
| `javascript.analysis_timeout_ms`, `javascript.max_heap_mb` | Phân tích nguồn tiếp theo / worker JavaScript mới |
| `wasm.command_timeout_ms` | Thao tác WABT tiếp theo |
| `native.matching_representation_bytes` | Yêu cầu `match_native_functions` tiếp theo |

### Thứ tự ưu tiên và vòng đời (Precedence and lifetime)

Nơi cài đặt hỗ trợ đối số rõ ràng cho mỗi lệnh gọi, thứ tự ưu tiên là:

1. Đối số rõ ràng của bên gọi.
2. Ghi đè thời gian chạy của thao tác được chấp nhận.
3. Cấu hình khởi chạy hợp lệ, nơi cài đặt đó có hạt giống môi trường.
4. Giá trị mặc định tích hợp sẵn khi cấu hình khởi chạy vắng mặt.

Một giá trị khởi chạy không hợp lệ sẽ không tự động chuyển về mặc định tích hợp. Bảo toàn các đối số bên gọi bị bỏ qua trong Evidence: việc giải quyết giá trị mặc định nội bộ không được viết lại yêu cầu như thể bên gọi đã cung cấp nó. `null` là giá trị không giới hạn rõ ràng cho `ghidra.decompile_timeout_seconds`; nó không phải là sự thay thế cho một giá trị chưa xác định hoặc bị bỏ sót ở nơi khác.

Lệnh ghi không mở rộng công việc đang hoạt động, không khởi chạy lại provider, không xóa quá trình dọn dẹp được lưu giữ hoặc làm mất hiệu lực của Evidence thành công.

Ghi đè thuộc về một runtime REA, tồn tại qua việc đóng/mở mục tiêu và biến mất khi REA thoát. Các kết nối dùng chung runtime đó sẽ chia sẻ các ghi đè của nó. Không có tệp, môi trường xung quanh, đăng ký hoặc phần mềm upstream nào bị chỉnh sửa. Ghi đè có phạm vi toàn runtime, không giới hạn trong một mục tiêu hoặc một kết nối riêng lẻ.

### Lỗi và thử lại (Failure and retry)

Lỗi tiếp nhận ASAR cung cấp `details.limit` với cài đặt chính xác, giá trị hiệu quả, đơn vị, ranh giới, phạm vi, điểm áp dụng, yêu cầu khởi động lại và khi đã biết, số byte bắt buộc và nguồn giá trị. Tác tử có thể gọi ngay lập tức:

```json
{
  "changes": [
    { "setting": "asar.max_header_bytes", "action": "set", "value": 33554432 }
  ]
}
```

Sau đó thử lại cùng yêu cầu toàn kho lưu trữ sau khi công việc và việc dọn dẹp đã ổn định. Không cần kiểm tra sơ bộ hoặc tập hợp con nguồn. Hàm setter không tự động thử lại. Nếu một giá trị rõ ràng cho mỗi lệnh gọi gây ra lỗi, hãy sửa đối số đó; nâng cao mặc định thời gian chạy không thể ghi đè lên nó.

Chủ sở hữu tài nguyên xây dựng `details.limit` từ sổ đăng ký cộng với ngân sách mà nó thực sự thực thi. Metadata bao gồm `agent_editable: true`, `scope: "rea-runtime"` và `rea_restart_required: false`. Các trường tùy chọn `required_value`, `source` và `parameter` chỉ được cung cấp khi đã biết hoặc có thể áp dụng. Không suy luận nguồn vì một giá trị tình cờ bằng giá trị mặc định. Sao chép lỗi, quy kết, biên tập và phân phối phải bảo toàn các dữ kiện này. Biện pháp khắc phục con người có thể đọc được giải thích cùng một lỗi; client không cần phải phân tích câu văn đó để xác định cài đặt có thể chỉnh sửa.

Chính sách bộ nhớ đệm là một phần của việc thực thi. Phân tích có thể tái sử dụng thành công có thể tồn tại sau khi cập nhật ngân sách. Các lỗi dịch ngược Ghidra tạm thời vẫn không được lưu trong bộ nhớ đệm để thời hạn được nâng cao có thể phục hồi trên cùng một Program. Kiểm tra tiếp nhận vẫn áp dụng cho dữ liệu được tái sử dụng: hạ thấp giới hạn ASAR phải từ chối tiêu đề quá khổ được lưu trong bộ nhớ đệm. Cả việc nâng giới hạn lẫn việc xóa lỗi phân tích tạm thời đều không cho phép bỏ qua việc dọn dẹp hoặc thay đổi danh tính của phiên provider.

### Cấu hình khởi chạy không hợp lệ (Invalid launch configuration)

Xác thực khởi chạy có hai ranh giới liên quan. Cài đặt được xác thực bằng phân tích cấu hình toàn cục có thể ngăn REA khởi động; một setter MCP không thể sửa chữa một máy chủ không chạy. Sửa môi trường khởi chạy hoặc đăng ký, sử dụng `rea doctor` cho các kiểm tra cấu hình của nó và khởi động lại REA.

Xác thực cục bộ của provider có thể để MCP khả dụng. Ví dụ: một `REA_JADX_OPERATION_TIMEOUT_MS` không hợp lệ có thể kiểm tra được với `source: "invalid"`, một `configuration_error` và `value` cùng `launch_value` bị bỏ qua. Một ghi đè thời gian chạy hợp lệ sẽ sửa thời gian chờ JADX hiệu quả mà không cần chỉnh sửa hạt giống khởi chạy đó. Sau đó, việc kiểm tra sẽ báo cáo giá trị thời gian chạy trong khi vẫn giữ lại lỗi khởi chạy. Reset sẽ hiển thị lại trạng thái khởi chạy không hợp lệ; nó không sửa chữa hoặc thay thế nó bằng giá trị mặc định tích hợp. Tính khả dụng và thực thi của provider phải giải quyết cùng một cài đặt hiệu quả để tính sẵn sàng thay đổi cùng với việc sửa chữa và reset. `REA_NATIVE_MATCHING_REPRESENTATION_BYTES` được xác thực theo cùng cách, chỉ bởi `match_native_functions`.

## Giữ sự trung thực về mức trần cố định (Keep a fixed ceiling honest)

Các [ràng buộc](https://github.com/morluto/rea/blob/main/src/server/analysisSettingsConstraints.ts) riêng biệt của việc kiểm tra mô tả các kiểm soát do người vận hành sở hữu, bên ngoài và cố định. REA có thể báo cáo ngân sách phản hồi được cấu hình và giới hạn heap cha của nó; thời hạn của client và bộ đệm nhận vẫn rõ ràng là chưa xác định. Đây không phải là các mục được chấp nhận bởi setter thời gian chạy. Nâng yêu cầu heap của worker không thể nâng heap cha và việc nâng thời hạn của provider không thể làm cho client chờ đợi lâu hơn.

Nếu trần phải cố định, hãy nêu rõ trong lỗi. Đặt tên giới hạn và giá trị hiệu quả, thông báo rằng bản phát hành này không có ghi đè thời gian chạy và nêu tên những gì không nâng nó lên. Không bịa ra một cài đặt.

Các giá trị JSON dừng ở `MAX_JSON_DEPTH` (1000) trong [`jsonValue.ts`](https://github.com/morluto/rea/blob/main/src/domain/jsonValue.ts): "Giá trị JSON vượt quá độ sâu lồng nhau tối đa là 1000. Bản phát hành này không có ghi đè thời gian chạy."

## Văn bản phục hồi (Recovery text)

Chuỗi ký tự mà bên gọi nhìn thấy bao gồm:

- tên giới hạn hoặc cài đặt
- giá trị hiệu quả và đơn vị
- nguồn khi đã biết (`default`, `setting`, `runtime` hoặc `request`)
- mặc định, tối đa và mức sàn khi cài đặt có
- hành động tiếp theo, bao gồm cả nơi thiết lập nó ("trong môi trường bắt đầu REA")
- liệu có cần khởi động lại hay không
- những gì thay đổi không làm tăng lên

Phân biệt cấu hình không hợp lệ với ngân sách hợp lệ bị cạn kiệt bởi công việc. Bảo toàn lỗi cấu hình và danh tính cài đặt của nó; áp dụng [ranh giới xác thực khởi chạy](#cấu-hình-khởi-chạy-không-hợp-lệ) khi mô tả liệu việc sửa chữa trực tiếp có khả thi hay không.

## Tác tử cần làm gì (What agents do)

Sử dụng `details.limit` để cập nhật thời gian chạy trực tiếp khi nó đặt tên một cài đặt có thể chỉnh sửa. Làm theo `remediation.action` cho các kiểm soát cho mỗi lệnh gọi, cố định và bên ngoài. Chỉ kiểm tra cài đặt khi thiếu ranh giới hoặc chi tiết ứng dụng. Chỉ khởi động lại khi nó nói giá trị được đọc khi khởi động. Một câu nói rằng một kiểm soát không nâng giới hạn không phải là yêu cầu thay đổi nó.

- Mất kết nối MCP: chỉ thử lại lệnh gọi một lần duy nhất khi nó an toàn để lặp lại hoặc biết chắc là chưa bắt đầu. Nếu nó có thể đã làm thay đổi mục tiêu, hoặc tiếp tục thất bại, hãy thông báo cho người dùng và yêu cầu họ khởi động lại client MCP. Không bắt đầu bằng `rea doctor` ([#6353](https://github.com/morluto/rea/pull/6353)).
- REA không thể bắt đầu kết nối của nó: thử lại một lần. Nếu thất bại một lần nữa, hãy báo cho người dùng và yêu cầu họ khởi động lại client MCP, sau đó chạy `rea doctor`. Không thử lại một lần khởi động trả về kết quả giống hệt không đổi: báo cho người dùng rằng REA không thể khởi động, yêu cầu họ khởi động lại client MCP, sau đó chạy `rea doctor`.
- Lỗi lệnh tiến trình bất ngờ: không lặp lại lệnh. Yêu cầu người dùng kiểm tra các tệp đầu vào, sau đó chạy `rea doctor`.
- `cleanup_incomplete`: thao tác không thể thử lại. Thử lại chính chủ sở hữu dọn dẹp đó, được nêu tên trong `details.resources`, và chờ cho đến khi nó xác nhận giải phóng trước khi tiếp nhận công việc mới trên các tài nguyên đó ([#6357](https://github.com/morluto/rea/pull/6357)).
- `resource_constraint` cho dung lượng, khi `retryable` là true: đợi cho việc tiếp nhận đang hoạt động kết thúc, sau đó thử lại trong giới hạn được báo cáo. Không khởi động lại REA để giải phóng vị trí.
- `configuration_invalid`: khi MCP vẫn khả dụng và lỗi thuộc về một cài đặt cục bộ của provider được tuyển chọn, hãy kiểm tra và đặt một ghi đè thời gian chạy hợp lệ. Nếu không, hãy sửa cài đặt có tên trong môi trường bắt đầu REA, sau đó khởi động lại REA. Chạy `rea doctor` để kiểm tra `runtime-configuration:`. Chạy lại `rea setup` cho client đó khi cài đặt nằm trong danh sách cho phép đăng ký, sau đó kết nối lại để tiến trình mới nhìn thấy nó.
- `invalid_request`: sửa các trường được báo cáo từ lược đồ hiện tại hoặc trợ giúp CLI. Không lặp lại lệnh gọi không thay đổi.

Cần phải khởi động lại để áp dụng môi trường khởi chạy hoặc đăng ký đã thay đổi. Các cài đặt thời gian chạy được tuyển chọn ở trên không yêu cầu khởi động lại REA. Đó không phải là bước đầu tiên cho việc mất MCP tạm thời, chờ dung lượng hoặc dọn dẹp được lưu giữ.

## Thêm một cài đặt thời gian chạy (Adding a runtime setting)

1. Xác định chủ sở hữu thực thi và ràng buộc tài nguyên thực tế. Quyết định xem bên gọi có cần đối số cho mỗi lệnh gọi, mặc định thời gian chạy, kiểm soát chỉ khởi chạy hay giới hạn cố định. Một mục đăng ký không được quảng bá quyền hạn mà REA thiếu.
2. Xác định kiểu giá trị và ranh giới số học có tên với chủ sở hữu miền. Thêm ID và metadata vào `analysisSettings.ts` và `analysisSettingDefinitions.ts`. Tái sử dụng các ranh giới đó trong xác thực thời gian chạy, phân tích khởi chạy và metadata lỗi. Chọn điểm áp dụng và liên kết provider/thao tác từ các consumer thực tế; một kiểm soát dùng chung không cần áp dụng cho mọi provider.
3. Nếu có hạt giống khởi chạy, hãy thêm trình phân tích cú pháp của nó và xác thực cấu hình/doctor phù hợp. Quyết định xem một giá trị không hợp lệ có chặn runtime hay chỉ chặn provider. Thêm nó vào `SELECTED_REGISTRATION_SETTINGS` của thiết lập khi nó cần được chuyển tiếp; khả năng chỉnh sửa thời gian chạy không tự động truyền bá nó.
4. Nối reader dùng chung qua mọi đường dẫn cấu thành áp dụng. Giải quyết giá trị hiệu quả bên trong thao tác được chấp nhận, tôn trọng các đối số rõ ràng và bảo toàn sự bỏ sót của chúng trong Evidence. Giữ các kiểm tra tính sẵn sàng nhất quán với việc thực thi. Một yêu cầu heap của worker không được mô tả là giới hạn bộ nhớ tiến trình được đo lường.
5. Thực thi giá trị tại ranh giới tài nguyên của nó và xây dựng các dữ kiện giới hạn có cấu trúc ở đó. Bảo toàn các dữ kiện đó thông qua phép chiếu lỗi dùng chung và các bản sao. Kiểm tra việc tái sử dụng bộ nhớ đệm, mức tiếp nhận hạ thấp, lỗi tạm thời và quá trình dọn dẹp được lưu giữ theo cùng các quy tắc sở hữu.
6. Cập nhật các hợp đồng công khai và hướng dẫn bị ảnh hưởng, sau đó tạo lại danh mục và metadata kỹ năng đóng gói bằng các lệnh kho lưu trữ. Giữ các kiểm soát cố định và thuộc sở hữu bên ngoài trong các ràng buộc kiểm tra riêng biệt, không nằm trong sổ đăng ký có thể chỉnh sửa. Xác thực cả ngữ nghĩa kho lưu trữ và đường dẫn phục hồi công khai.

## Kiểm thử và xác minh (Tests and verification)

Bao phủ giá trị mặc định chưa đặt, các ghi đè được chấp nhận, các giá trị sàn/tối đa bị từ chối và cú pháp không hợp lệ thông qua trình phân tích cú pháp sở hữu và ranh giới cấu hình. [`regexMatchingDeadline.test.ts`](https://github.com/morluto/rea/blob/main/src/config/regexMatchingDeadline.test.ts) là một ví dụ. Khẳng định mã lỗi, ID cài đặt, giá trị, đơn vị, nguồn gốc, điểm áp dụng và hành vi kết quả. Kiểm tra xem các adapter có cung cấp hướng dẫn phục hồi chuẩn hay không, với các khẳng định độc lập cho các dữ kiện cơ bản của nó. Tránh đóng băng văn bản hoặc suy ra tất cả các giá trị mong đợi từ cùng một sổ đăng ký được triển khai sử dụng; cả hai đều có thể che giấu lỗi sai. Xem [thiết kế kiểm thử](../testing.md#test-design-and-pruning) cho chính sách assertion.

Các kiểm tra thời gian chạy bao gồm xác thực nguyên tử, ghi cũ, reset, cách ly thời gian chạy, bản chụp thao tác được chấp nhận và mức ưu tiên đối số rõ ràng. Việc chấp nhận MCP công khai phải từ chối một ASAR quá khổ, nâng mức tiếp nhận và thử lại thành công cùng một kho lưu trữ hoàn chỉnh trên cùng một kết nối. Hạ thấp mức tiếp nhận cũng phải từ chối các tiêu đề được lưu trong bộ nhớ đệm trước đó. Trình xác minh thời gian chờ Ghidra thực tế kiểm tra sự phục hồi sau khi cập nhật trực tiếp mà không cần thay thế phiên của nó. Đồng thời bao gồm cấu hình khởi chạy cục bộ của provider không hợp lệ, tính khả dụng của việc sửa chữa/reset, đường dẫn thành phần mặc định và tùy chỉnh, cùng việc bảo toàn các giới hạn có cấu trúc thông qua các bản sao lỗi. Các cổng được sử dụng để xếp chồng công việc và cập nhật cần có xác nhận giới hạn và giải phóng/kết hợp trong `finally`.

[Báo cáo xác minh](../verification/runtime-analysis-settings.md) tách biệt các kiểm tra MCP công khai, độ bao phủ provider mô phỏng, thực thi Ghidra thực tế và chi phí tác vụ được đo lường. Nó không xác lập hành vi của provider thực tế cho mọi cài đặt có thể chỉnh sửa hoặc đo từ xa của worker đang hoạt động.

## Đồ thị mô-đun độc lập của Bun (Bun standalone module graphs)

`MAX_BUN_MODULE_COUNT` chấp nhận tối đa 100.000 bản ghi mô-đun. `MAX_BUN_RETAINED_BYTES` chấp nhận tối đa 256 MiB payload đồ thị được tuần tự hóa và giới hạn riêng metadata mục nhập được lưu giữ. Công việc đọc metadata vật lý cũng được giới hạn ở 256 MiB cho mỗi lần chuẩn bị, bao gồm cả các phạm vi lặp lại chưa được lưu trong bộ nhớ đệm. Một bộ nhớ đệm phạm vi chính xác 256 KiB / 1.024 mục tránh việc khuếch đại toàn trang. Cây đường dẫn logic kết hợp chấp nhận tối đa 250.000 tiền tố riêng biệt; điều này được kiểm tra trước khi thêm các nút, với các tiền tố dùng chung được tính một lần. Ngân sách mở rộng hiện vật dùng chung cũng giới hạn tổng độ dài của các thành viên được tham chiếu. Sự từ chối đặt tên ranh giới. Không có ghi đè thời gian chạy; bộ nhớ máy chủ bổ sung hoặc `NODE_OPTIONS` không làm tăng các mức trần này. Sử dụng fixture producer nhỏ hơn hoặc reader được xem xét riêng cho các đồ thị lớn hơn. Không có tuyên bố trích xuất từng phần khi thất bại tiếp nhận.
