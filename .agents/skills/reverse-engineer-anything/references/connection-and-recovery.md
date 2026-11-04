# Khôi phục kết nối và tác vụ (Connection and task recovery)

Chỉ đọc tài liệu hướng dẫn này khi gặp phải điểm tắc nghẽn quan sát được về kết nối, lệnh, phê duyệt hoặc provider. Luôn ghi nhớ mục tiêu và câu hỏi của người dùng. Giải thích điều gì đã ngăn cản thao tác và bước đi hữu ích tiếp theo; sau khi sửa chữa, hãy tiếp tục cuộc điều tra.

Các lệnh dưới đây sử dụng CLI `rea` đã được cài đặt. Nếu chưa có sẵn, hãy sử dụng cách gọi gói trong `SKILL.md` khi việc tải REA được cho phép. Tuân thủ các lược đồ của server đã kết nối và phần trợ giúp của CLI đã cài đặt thay vì giả định nhánh main và bản phát hành cũ hơn cung cấp các đối số giống nhau. Tiền tố tên công cụ thuộc về phía client: hãy tìm thao tác REA chuẩn trong danh sách công cụ thực tế và liên kết server của nó trước khi kết luận rằng nó bị thiếu. Một tiền tố chung đơn thuần không chứng minh ký tự đại diện phê duyệt áp dụng cho server nào.

## Chọn biện pháp khắc phục từ lỗi quan sát được (Choose the remedy from the observed failure)

| Quan sát | Ý nghĩa và bước tiếp theo |
| --- | --- |
| Lệnh gọi công cụ bị từ chối bởi chính sách phê duyệt của client (ví dụ: `declined by policy` hoặc `interactive approval required`) | Chính sách client chặn lệnh gọi. REA không có cờ cấp quyền theo từng lệnh gọi. Xem [Phê duyệt công cụ Codex tách biệt với tính sẵn sàng của REA](#codex-tool-approval-is-separate-from-rea-readiness). |
| Không tìm thấy công cụ REA hoặc danh sách công cụ trống | Server MCP chưa được đăng ký hoặc chưa khởi động. Xem [REA được cài đặt dưới dạng plugin của client](#rea-installed-as-a-client-plugin) hoặc [Sửa chữa đăng ký khi cần thiết](#repair-registration-when-needed). |
| Kết nối MCP bị ngắt giữa chừng (`connection closed`, `broken pipe`, `process exited`) | Tiến trình server MCP bị dừng đột ngột hoặc sập. Chỉ thử lại lệnh gọi một lần duy nhất nếu an toàn. Nếu tiếp tục thất bại, yêu cầu người dùng khởi động lại client MCP. |
| Host hết thời gian chờ (`timed out awaiting tools/call`) mà không có phản hồi từ REA | Hết hạn thời gian chờ phía client trước khi server hoàn thành. Xem [Phân biệt thời hạn giữa client và server](#distinguish-client-and-server-deadlines). Không lặp lại lệnh gọi giống hệt. |
| `resource_constraint` với `details.resource: "transport"` | Dữ liệu phản hồi vượt quá giới hạn truyền tải của khung MCP. Dữ liệu phân tích vẫn được lưu giữ trong phiên. Sử dụng `inspect_analysis_view` với `parent_evidence_id` hoặc gọi `export_evidence_bundle`. |
| `cleanup_incomplete` | Một thao tác trước đó chưa dọn dẹp xong tài nguyên sở hữu. Đợi vài giây để việc dọn dẹp ổn định rồi thử lại, hoặc khởi động lại client MCP nếu tài nguyên bị khóa cứng. |
| `configuration_invalid` | Tham số cấu hình hoặc biến môi trường không hợp lệ. Đọc `details.message` và sửa đúng theo tài liệu. |
| `provider_unavailable` cho IDA, Ghidra, JADX hoặc Kuna | Engine tương ứng chưa được cài đặt, thiếu đường dẫn tuyệt đối hoặc không đáp ứng điều kiện tiên quyết (ví dụ: Java 17+, Python, license). |

## Phân biệt thời hạn giữa client và server (Distinguish client and server deadlines)

Thông báo của máy chủ như `timed out awaiting tools/call` mà không có kết quả từ REA có nghĩa là thời hạn của client đã hết hạn trước khi có kết quả trả về. Điều này không chứng minh rằng REA đã hoàn thành, đã hủy thành công hoặc đã giải phóng tất cả tài nguyên. Không lặp lại cùng một phân tích chưa được thay đổi. Chỉ kiểm tra trạng thái phiên/provider sẵn có và Evidence được lưu giữ khi thực sự hữu ích; một `binary_session` không hoạt động (idle) không phải là bằng chứng cho thấy mọi thao tác không phụ thuộc mục tiêu đã dừng lại. Tuyệt đối không bao giờ bịa ra một bản ghi Evidence bị thiếu.

Khi phân tích các tệp nhị phân lớn với Ghidra hoặc JADX, hãy nâng mức thời hạn bằng `update_analysis_settings` (`ghidra.startup_timeout_ms` hoặc `jadx.operation_timeout_ms`) trước khi thực hiện lần thử tiếp theo, và đảm bảo thời gian chờ của MCP client đủ lớn để chấp nhận thời hạn đó.

## Bậc thang khôi phục (Recovery ladder)

Tuân thủ hướng dẫn khắc phục được trả về. Chỉ thay đổi một trường yêu cầu hoặc thiết lập `REA_*` khi văn bản hướng dẫn khuyến nghị rõ ràng thay đổi đó. Khởi động lại REA khi nó thông báo giá trị được đọc tại thời điểm khởi động. Một ghi chú cho biết rằng một điều khiển không nâng giới hạn không phải là khuyến nghị để thay đổi nó. Không tự ý bịa ra thiết lập. Bộ nhớ máy chủ và `NODE_OPTIONS` không làm tăng hạn mức tiếp nhận của REA trừ khi hướng dẫn khắc phục đó nêu rõ.

- Kết nối MCP bị ngắt: chỉ thử lại lệnh gọi đó một lần duy nhất khi nó an toàn để lặp lại hoặc biết chắc là chưa bắt đầu.
- REA không thể bắt đầu kết nối: thử lại một lần, sau đó thông báo cho người dùng và yêu cầu họ khởi động lại client MCP.
- Đối với `cleanup_incomplete`, giới hạn dung lượng có thể thử lại, hoặc `configuration_invalid`: làm theo từng bậc trong hướng dẫn trước khi yêu cầu khởi động lại.
- Đối với đăng ký do setup quản lý: chạy `rea doctor` khi REA không thể khởi động hoặc các công cụ vẫn thiếu sau khi khởi động lại.

## REA được cài đặt dưới dạng plugin của client (REA installed as a client plugin) <a id="rea-installed-as-a-client-plugin"></a>

Một plugin Claude Code hoặc Codex sở hữu server `rea` riêng của nó và skill này. Các kiểm tra `--client` và `--skill` của `rea doctor` cũng như lệnh `rea setup` chỉ đọc và ghi trạng thái do setup quản lý, do đó đối với bản cài đặt chỉ có plugin, chúng sẽ báo cáo `missing`, và việc áp dụng setup sẽ thêm một server `rea` thứ hai. Nếu skill này được nạp từ một plugin, hoặc client liệt kê một plugin `rea`, không chuẩn bị kế hoạch setup cho các công cụ bị thiếu. Thay vào đó:

- Kiểm tra xem plugin đã được cài đặt và kích hoạt trong client hay chưa.
- Khởi động lại client để plugin nạp lại server MCP và đăng ký công cụ.
- Nếu plugin bị lỗi, hãy gỡ bỏ và cài đặt lại plugin theo tài liệu chính thức của client đó.

## Sửa chữa đăng ký khi cần thiết (Repair registration when needed) <a id="repair-registration-when-needed"></a>

Đối với Codex hoặc Antigravity:

```bash
rea doctor --client codex --json
rea setup --client codex --dry-run --json
```

Sử dụng ID client được hỗ trợ hiện tại: `claude_code`, `claude_desktop`, `codex`, `cursor`, `gemini_cli`, `windsurf`, `devin`, `opencode`, `antigravity`, `copilot_cli`, `commandcode`, `qwen_code`, `vscode`, `trae`, `grok_build`, `omp`, `pi`, `hermes`, `qoder`, `droid`, `zcode`, `dsh`, `freebuff`, hoặc `grok_bot`. Nếu không rõ, hãy kiểm tra `doctor --json` trước khi chọn client. Luôn chạy với `--dry-run` trước để xem kế hoạch sửa chữa đăng ký chỉ đọc (paths, backups và skill replacement) trước khi áp dụng các thay đổi được ủy quyền.

## Phê duyệt công cụ Codex tách biệt với tính sẵn sàng của REA (Codex tool approval is separate from REA readiness) <a id="codex-tool-approval-is-separate-from-rea-readiness"></a>

Các công cụ REA không có cơ chế bắt tay cấp quyền riêng. Codex hoặc agent client vẫn có thể yêu cầu phê duyệt vì một thao tác tạo ra tài nguyên sở hữu, khởi chạy tiến trình trợ giúp hoặc có hiệu ứng dọn dẹp. Một công cụ được quảng bá hoặc kết quả doctor thành công không ghi đè chính sách client đó. Cờ `workspace-write` đơn thuần không giải quyết được việc từ chối phê duyệt MCP, và `approval_policy = "never"` không có nghĩa là mọi công cụ MCP đều được phê duyệt trước.

Khi phê duyệt tương tác khả dụng, hãy sử dụng quy trình phê duyệt của client. Nếu một lệnh gọi bị từ chối, hãy giải thích rõ ràng công cụ nào cần được phê duyệt và mục đích của nó, sau đó yêu cầu người dùng phê duyệt hoặc tiếp tục thông qua CLI thay thế.

## Sửa đổi giới hạn REA có thể chỉnh sửa (Repair an editable REA limit)

Nếu `details.limit` đặt tên cho một thiết lập có thể chỉnh sửa và ranh giới của nó, hãy gọi trực tiếp `update_analysis_settings` với một mục `changes` chứa `setting`, `action: "set"` và `value`. Việc gọi `inspect_analysis_settings` là tùy chọn khi các dữ kiện đó đã hiện diện. Thử lại cùng đầu vào sau khi công việc và việc dọn dẹp đã ổn định. Các giá trị mặc định của runtime chỉ ảnh hưởng đến công việc mới; các đối số rõ ràng theo từng lệnh gọi sẽ được ưu tiên và phải được sửa trực tiếp khi được chọn. Thao tác reset sẽ khôi phục các giá trị lúc khởi chạy.

## Tiếp tục thông qua CLI đã cài đặt (Continue through an installed CLI)

Kiểm tra môi trường lệnh chỉ khi trình khởi chạy gặp lỗi:

```bash
command -v rea
command -v node
node --version
rea --version
```

Sử dụng trình khởi chạy công khai tại đường dẫn cài đặt đã biết của nó với một bản Node được hỗ trợ sẵn có trên PATH khi shell con bị mất cài đặt tiền tố. Tránh các điểm vào nội bộ `dist/cli.js` hoặc các thay đổi chuỗi công cụ toàn cục. MCP có thể có trình khởi chạy hợp lệ của riêng nó ngay cả khi shell không thể giải quyết lệnh `rea`.

Đối với cây mã JavaScript hoặc gói ASAR do người dùng cung cấp:

```bash
npx -y rea-agents@latest analyze-javascript-application /absolute/path/to/app --json
```

Lệnh này chạy độc lập mà không cần đăng ký MCP hoặc engine native, trực tiếp trả về bản ghi Evidence hoàn chỉnh dưới dạng JSON.
