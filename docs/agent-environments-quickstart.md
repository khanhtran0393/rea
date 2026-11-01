# Xây dựng môi trường Học tăng cường (RL) với REA

REA biến một phiên làm việc ứng dụng web đã ghi lại thành một môi trường có thể chạy được để huấn luyện và kiểm thử các AI agent. Một agent có thể thực hành các tác vụ như tìm lỗi, tạo thư mục hoặc chỉnh sửa bản ghi mà không cần chạy ứng dụng gốc.

REA xây dựng các công cụ mà agent gọi, dữ liệu cục bộ mà các công cụ đó sử dụng và một bài kiểm tra để xác nhận môi trường khởi động và phản hồi. Nó hỗ trợ các thay đổi khi bản ghi xác nhận hiệu ứng của chúng. Sau đó, bạn có thể sử dụng môi trường này với framework Học tăng cường (RL) của mình để đặt mục tiêu nhiệm vụ, kiểm tra phần thưởng (rewards) và chạy quá trình huấn luyện.

## Trước khi bạn bắt đầu

Cài đặt [REA và tích hợp agent của nó](installation.md). Để ghi lại một ứng dụng, bạn cũng cần một phiên bản thử nghiệm đang chạy và một trình duyệt thuộc dòng Chrome. Hãy chọn một phiên bản mà agent được phép tạo, sửa và xóa dữ liệu. REA ghi lại ứng dụng bạn cung cấp; bạn khởi động nó và chuẩn bị dữ liệu thử nghiệm.

Để chạy môi trường đã xuất, hãy cài đặt [AgentEnv framework](https://github.com/scaleapi/agentenv-framework#install) và khởi động Docker. [Kiểm tra tính tương thích](testing.md#exported-environment-bundles) của REA sử dụng AgentEnv phiên bản `0.9.1334` với Python 3.12.

Có hai cách để xây dựng môi trường: yêu cầu agent của bạn ghi lại ứng dụng, hoặc bắt đầu từ một bản ghi bạn đã có sẵn.

## 1. Ghi lại và xây dựng cùng agent của bạn

Cung cấp cho agent của bạn URL ứng dụng, các thao tác bạn cần, đường dẫn tệp thực thi của trình duyệt và thư mục đầu ra. Đối với ứng dụng có thư mục, hãy điều chỉnh lời nhắc sau:

```text
Hãy sử dụng REA để xây dựng môi trường AgentEnv từ ứng dụng web thử nghiệm của tôi tại
http://127.0.0.1:3000. Bao gồm việc liệt kê, tạo, đổi tên và xóa thư mục.
Bạn có thể tạo và xóa các thư mục thử nghiệm. Hãy giữ nguyên dữ liệu hiện có.

Đăng nhập với tư cách admin bằng mật khẩu trong biến APP_PASSWORD. Sử dụng trình duyệt tại
/duong/dan/tuyet/doi/toi/chromium. Lưu môi trường vào
/duong/dan/tuyet/doi/toi/my-env, thư mục này hiện chưa tồn tại.

Ghi lại phiên làm việc, kiểm tra xem môi trường hỗ trợ những thao tác nào và xuất nó ra.
Báo cáo bất kỳ thao tác nào không được hỗ trợ và những gì còn thiếu.
```

Thay thế URL, tài khoản, đường dẫn và các thao tác thư mục bằng thông tin của bạn. Thiết lập `APP_PASSWORD` trong môi trường khởi động MCP server của REA để REA có thể sử dụng mà không cần đưa mật khẩu vào lời nhắc.

Agent của bạn sẽ kiểm tra ứng dụng, ghi lại phiên làm việc, kiểm tra kết quả và xuất một thư mục chứa môi trường có thể chạy được. Nó báo cáo thao tác nào hoạt động và thao tác nào cần thêm bằng chứng. Ứng dụng chỉ được sử dụng trong quá trình ghi; môi trường đã xuất sẽ chạy từ dữ liệu đã lưu.

Sau khi agent xuất thư mục, chuyển tới bước [chạy môi trường](#3-chay-moi-truong). Phần tiếp theo là giải pháp thay thế cho bản ghi đã có từ trước.

## 2. Hoặc xây dựng từ một bản ghi đã lưu

Tệp HAR ghi lại các yêu cầu và phản hồi mạng của trình duyệt. Xuất một tệp kèm phần thân phản hồi API từ bảng mạng của trình duyệt, hoặc sử dụng bản ghi bạn đã có sẵn. Để bao gồm các thao tác tạo, sửa và xóa, hãy ghi lại các hành động đó và các lần đọc hiển thị kết quả của chúng.

Xây dựng môi trường vào một thư mục mới hoặc trống:

```sh
rea environment-build ./session.har ./my-env --name my-env --json
```

Bạn cũng có thể truyền JSON Bằng chứng (Evidence) được lưu bởi các lệnh `capture-browser-scenario` hoặc `inspect-web-network-capture` của REA. Ở đây, Bằng chứng có nghĩa là bản ghi đã lưu của REA cùng thông tin nguồn của nó. Quá trình xây dựng chấp nhận các tệp HAR và các bản ghi JSON đó. Nó đọc dữ liệu cục bộ và ghi thư mục môi trường mà không cần liên hệ với ứng dụng.

Thêm cờ `--dry-run` để xem lại báo cáo xây dựng mà không ghi thư mục. Đối với bản dựng REA chưa phát hành, hãy sử dụng `--package-tarball` với gói đã đóng gói để môi trường đã xuất chạy cùng một triển khai.

Báo cáo xây dựng cho bạn biết môi trường hỗ trợ những gì:

| Trường báo cáo | Nội dung kiểm tra |
| :--- | :--- |
| `state` | Môi trường có thể trả lời từ trạng thái cục bộ hay sử dụng phát lại phản hồi đã ghi. |
| `coverage.reads` | Những thao tác đọc nào sử dụng trạng thái và thao tác nào phát lại câu trả lời đã ghi. |
| `coverage.writes` | Những thao tác tạo, sửa và xóa nào được hỗ trợ. Mỗi thao tác bị giữ lại kèm theo một lý do và gợi ý ghi lại. |
| `warnings` | Nội dung đã ghi hoặc đường dẫn cục bộ cần xem lại trước khi chia sẻ thư mục. |

Xây dựng thành công có nghĩa là thư mục đã được tạo. Hãy kiểm tra báo cáo để xác nhận nó hỗ trợ các thao tác mà tác vụ của bạn cần. `coverage: null` có nghĩa là bản dựng không thể chuẩn bị trạng thái cho môi trường này và nó sẽ sử dụng chế độ phát lại (replay).

## 3. Chạy môi trường

Dù agent của bạn đã xuất thư mục hay bạn tự xây dựng từ tệp HAR, hãy chạy nó với AgentEnv:

```sh
agent-env run ./my-env
```

Sử dụng đường dẫn thư mục mà agent báo cáo nếu có sự khác biệt. Lần chạy đầu tiên sẽ xây dựng image container, vì vậy nó cần quyền truy cập vào nguồn image và package.

REA bao gồm một bài kiểm tra cơ bản, gọi là bài kiểm tra smoke. Nó kiểm tra số lượng công cụ dự kiến và khi có công cụ đọc phù hợp, nó sẽ gọi công cụ đó và kiểm tra trạng thái phản hồi. Vượt qua bài kiểm tra này cho thấy môi trường đã khởi động và phản hồi trong thiết lập thử nghiệm. Nó chưa chấm điểm agent trên một tác vụ ứng dụng thực tế.

Thư mục chứa bản ghi, một manifest công cụ mô tả các thao tác khả dụng, một Dockerfile, cấu hình môi trường và bài kiểm tra. [Tài liệu tham khảo xuất bản](agent-environments.md#export-a-bundle) mô tả chi tiết từng tệp.

## 4. Thêm nhiệm vụ và kiểm tra phần thưởng

Bắt đầu với một nhiệm vụ mà các công cụ đã xuất hỗ trợ. Hãy đặt cho nó một mục tiêu rõ ràng và kết quả có thể kiểm tra được. Ví dụ: nếu việc tạo thư mục được hỗ trợ, hãy hỏi agent của bạn:

```text
Sử dụng môi trường đã xuất này để thêm một nhiệm vụ AgentEnv yêu cầu agent
tạo một thư mục có tên là Practice. Kiểm tra trạng thái môi trường đã lưu để
xác nhận thư mục tồn tại với tên đó, và sử dụng kiểm tra đó để tính điểm kết quả.
Đặt lại (reset) môi trường trước mỗi lần chạy để các lần thử bắt đầu với cùng dữ liệu.
Sử dụng định dạng nhiệm vụ và trình xác minh của AgentEnv.
```

Thao tác này yêu cầu agent hỗ trợ xây dựng nhiệm vụ và kiểm tra chấm điểm xung quanh môi trường mà REA đã tạo ra. Một nhiệm vụ truy xuất có thể kiểm tra câu trả lời dựa trên một giá trị đã ghi. Một nhiệm vụ làm thay đổi dữ liệu phải kiểm tra trạng thái kết quả. Khi trạng thái có sẵn, môi trường hỗ trợ các thao tác đặt lại, tải và đọc lại cho các lần chạy lặp lại.

Xem [tài liệu AgentEnv](https://www.agentenvframework.com/docs) để biết cấu hình nhiệm vụ, kết nối tác tử, tính điểm và thực thi. Kết nối các nhiệm vụ thu được vào quy trình huấn luyện RL đã chọn của bạn.

## Cải thiện một thao tác còn thiếu

Để REA hỗ trợ một thay đổi, bản ghi cần hiển thị những gì đã xảy ra trước và sau thay đổi đó. Khi ghi lại đối với ứng dụng thử nghiệm của bạn:

1. Đọc danh sách và một mục hiện có trước khi thực hiện thay đổi. Lặp lại các lần đọc ban đầu để hiển thị giá trị nào thay đổi độc lập.
2. Tạo một mục thử nghiệm, đọc lại nó và đọc danh sách trước lần thay đổi tiếp theo. Lặp lại với mục thứ hai và nội dung khác.
3. Chỉnh sửa từng mục thử nghiệm, sau đó đọc lại mục và danh sách. Để cách nhau hơn một giây giữa lúc tạo và lần sửa đầu tiên để hiển thị các thay đổi phụ thuộc thời gian.
4. Xóa các mục thử nghiệm, sau đó đọc lại chúng và đọc danh sách để xác nhận việc xóa.

Đồng thời ghi lại các bộ lọc, tìm kiếm, sắp xếp và phân trang nếu nhiệm vụ của bạn cần chúng.
Agent phải kiểm tra xem mọi bước ghi đã hoàn thành trước khi sử dụng.
[Quy trình ghi lại](https://github.com/morluto/rea/blob/main/.agents/skills/reverse-engineer-anything/references/environments.md)
có hướng dẫn chi tiết về trình duyệt, bao gồm thời gian chờ và xử lý các ID do ứng dụng gán.

Sử dụng gợi ý của báo cáo để quyết định nội dung cần ghi tiếp theo:

| Lý do | Bước tiếp theo |
| :--- | :--- |
| `unconfirmed` | Đọc mục đó và danh sách của nó sau khi thay đổi. |
| `no-state-target` | Kiểm tra mẫu đường dẫn và các chế độ xem đã ghi; ghi lại các lần đọc còn thiếu. Biểu mẫu tuyến đường có thể không được hỗ trợ làm mục tiêu đột biến. |
| `no-observation` | Đọc lại một mục đã tạo. Đối với việc sửa hoặc xóa, cũng đọc nó trước khi thay đổi. |
| `contradicted` hoặc `side-effect` | Kiểm tra sự bất đồng được báo cáo. Thao tác có thể cần hành vi mà REA chưa thể mô hình hóa. |

Xem [bảng lý do đầy đủ](agent-environments.md#build-a-bundle-in-one-step) cho các trường hợp khác. Xây dựng lại từ bản ghi mới và kiểm tra lại báo cáo. Một bản ghi mới thay thế đầu vào của bản dựng; việc kết hợp các phiên ghi riêng biệt không được hỗ trợ.

## So sánh với ứng dụng gốc

Để kiểm tra chắc chắn hơn, hãy ghi lại một chuỗi lệnh gọi riêng biệt đối với ứng dụng thực, bắt đầu từ cùng một dữ liệu thử nghiệm như bản ghi ban đầu. Đây là một đầu dò (probe). Khôi phục dữ liệu bắt đầu đó trước mỗi chuỗi đầu dò, sau đó so sánh các bản ghi đầu dò đã lưu với môi trường:

```sh
rea environment-compare ./my-env/envs/my-env/manifest.json \
  --evidence ./my-env/envs/my-env/evidence.json \
  --probe ./probe-create-edit-delete.json --probe ./probe-reads.json --json
```

Mỗi tệp đầu dò phải là JSON Bằng chứng (Evidence) từ REA. Việc so sánh chạy ngoại tuyến và báo cáo sự khác biệt trong phản hồi, các lệnh gọi không được hỗ trợ và các quan sát còn thiếu. Nó tính đến các khác biệt được hỗ trợ về ID, thời gian và các giá trị tạo ra được mô hình hóa. Sự tương đồng áp dụng cho các lệnh gọi mà các đầu dò đó kiểm tra.
Xem [tài liệu so sánh](agent-environments.md#measure-fidelity-against-the-application) để biết các quy tắc chính xác.

## Trước khi chia sẻ môi trường

Thư mục đã xuất bao gồm các phần thân phản hồi được lưu giữ và các đường dẫn nguồn cục bộ. Các header xác thực tiêu chuẩn được loại trừ, nhưng header tùy chỉnh, giá trị truy vấn và mã token trong thân phản hồi có thể vẫn còn. Hãy xem lại nội dung bạn định chia sẻ. Cảnh báo tên thông tin xác thực chỉ ra các giá trị đáng để xem xét; chúng không xóa các giá trị đó hoặc chứng minh rằng bản ghi không chứa thông tin xác thực.

Để phục vụ trực tiếp qua MCP, hành vi phát lại, kiểm soát trạng thái và giới hạn tài nguyên, xem [tài liệu môi trường](agent-environments.md).
[Mục lục triển khai](specs/agent-environments/README.md) liên kết các PR đóng góp và hồ sơ xác minh của chúng.
