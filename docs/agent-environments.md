# Môi trường Tác tử từ các Bản ghi Thu thập (Agent Environments)

Đối với môi trường đầu tiên, hãy bắt đầu với [hướng dẫn nhanh](agent-environments-quickstart.md): ghi lại một ứng dụng hoặc sử dụng phiên đã lưu, xây dựng và chạy môi trường, sau đó thêm tác vụ và kiểm tra phần thưởng. [Chỉ mục triển khai](specs/agent-environments/README.md) liên kết các PR đóng góp và hồ sơ xác minh của chúng. Trang này là tài liệu tham khảo chi tiết về lệnh và hành vi.

Sử dụng `rea environment-build` để chuyển một bản ghi thu thập thành gói môi trường tác tử (agent environment bundle):

```sh
rea environment-build ./capture.har --output ./env-bundle --json
rea environment-build ./recording.saz --output ./env-bundle --json
rea environment-build ./scenario.json --output ./env-bundle --json
```

## Xây dựng gói môi trường trong một bước

Để xây dựng trực tiếp một gói từ bản ghi mạng hoặc kịch bản trình duyệt:

```sh
rea environment-build ./capture.har --output ./env-bundle --json
```

Lệnh này phân tích cú pháp các mục nhập, trích xuất các đường dẫn URL duy nhất, tham số truy vấn, tiêu đề, nội dung yêu cầu/phản hồi, và tạo ra:
- `manifest.json`: Siêu dữ liệu mô tả môi trường, các endpoint được hỗ trợ, và các quy tắc khớp.
- `state.json`: Trạng thái ứng dụng ban đầu được quan sát từ bản ghi.
- `routes/`: Các phản hồi HTTP được ghi lại được định tuyến theo phương thức và đường dẫn.

## Từ một kịch bản trình duyệt (Browser Scenario)

Bạn cũng có thể xây dựng môi trường từ kết quả chạy kịch bản trình duyệt:

```sh
rea capture-browser-scenario ./scenario.json --output ./scenario-capture.json
rea environment-build ./scenario-capture.json --output ./env-bundle --json
```

Điều này chụp lại cả tương tác giao diện người dùng (DOM, ảnh chụp màn hình, bộ nhớ cục bộ) và lưu lượng mạng tương ứng.

## Manifest là gì

Tệp `manifest.json` trong gói môi trường chứa:
- `schema_version`: Phiên bản lược đồ môi trường tác tử (ví dụ: `1.0.0`).
- `name`: Định danh duy nhất cho môi trường.
- `description`: Mô tả ứng dụng và luồng được mô phỏng.
- `endpoints`: Danh sách tất cả các điểm cuối HTTP được hỗ trợ, bao gồm phương thức, mẫu URL, và các biến động.
- `initial_state`: Ảnh chụp trạng thái ban đầu của cơ sở dữ liệu hoặc kho lưu trữ phía máy khách.
- `evaluation_rules`: Các tiêu chí đánh giá cho tác tử hoàn thành nhiệm vụ.

## Ý nghĩa và giới hạn của việc phát lại (Replay)

- **Những gì phát lại làm**: Trả lại chính xác các phản hồi đã được quan sát trong quá trình ghi lại cho các yêu cầu khớp mẫu; mô phỏng độ trễ mạng thực tế; xác thực tính hợp lệ của tham số yêu cầu từ tác tử.
- **Những gì phát lại không làm**: Nó không chạy mã máy chủ phụ trợ gốc; nó không thực thi logic nghiệp vụ phức tạp ngoài những gì đã được ghi nhận hoặc khai báo trong trạng thái; các điểm cuối chưa ghi lại sẽ trả về lỗi `404 Not Found` hoặc phản hồi mô phỏng mặc định.

## Trạng thái mà bản ghi thu thập quan sát được

Bản ghi thu thập nắm bắt trạng thái ứng dụng tại thời điểm ghi:
- Dữ liệu người dùng, cấu hình phiên làm việc, mã thông báo giả lập.
- Cookies, `localStorage`, `sessionStorage`.
- Cấu trúc dữ liệu REST/GraphQL và phản hồi cơ sở dữ liệu tương ứng.

## Phục vụ một Manifest (Serve a Manifest)

Khởi chạy máy chủ phát lại môi trường tác tử cục bộ:

```sh
rea environment-serve ./env-bundle/manifest.json --port 8080
```

Tác tử của bạn có thể cấu hình `HTTP_PROXY` hoặc `BASE_URL` trỏ tới `http://127.0.0.1:8080` để tương tác trực tiếp với ứng dụng như thể nó đang chạy trên môi trường thực tế.

## Thẻ Môi trường (The Environment Card)

Mỗi gói môi trường bao gồm một tệp `card.json` hoặc phần `card` trong manifest:
- **Nhiệm vụ (Task)**: Mục tiêu tác tử cần đạt được (ví dụ: "Đăng nhập và cập nhật email trong hồ sơ").
- **Phần thưởng (Reward criteria)**: Điều kiện trạng thái hoặc yêu cầu mạng cần được đáp ứng để hoàn thành tác vụ.
- **Quan sát (Observation space)**: URL bắt đầu, thông tin xác thực ban đầu, trạng thái DOM có sẵn.
- **Hành động (Action space)**: Các lệnh API hoặc bước tương tác trình duyệt được phép.

## Phục vụ từ Trạng thái (Serve from the State)

Máy chủ môi trường hỗ trợ cập nhật trạng thái tương tác:
- Khi tác tử gửi yêu cầu `POST`, `PUT`, hoặc `DELETE`, máy chủ có thể cập nhật `state.json` trong bộ nhớ.
- Các yêu cầu `GET` tiếp theo sẽ phản ánh các thay đổi trạng thái này, tạo ra trải nghiệm phản hồi động chân thực cho tác tử.

## Xuất một Gói (Export a Bundle)

Xuất gói sang định dạng độc lập để chia sẻ hoặc lưu trữ:

```sh
rea environment-export ./env-bundle --archive ./agent-env.tar.gz
```

## Đo lường độ trung thực so với ứng dụng thực tế

So sánh hành vi của môi trường tác tử với ứng dụng trực tiếp:

```sh
rea environment-compare ./env-bundle --live-target https://app.example.com --json
```

Báo cáo so sánh bao gồm:
- **Độ bao phủ điểm cuối (Endpoint coverage)**: Tỷ lệ các tuyến đường được mô phỏng so với tuyến thực tế.
- **Độ nhất quán của lược đồ (Schema drift)**: Bất kỳ sự khác biệt nào về cấu trúc phản hồi JSON.
- **Độ chênh lệch tiêu đề (Header divergence)**: Các tiêu đề bảo mật hoặc bộ đệm khác nhau.

## Xác minh tính tương thích

Đảm bảo rằng môi trường tác tử tương thích với các khung kiểm thử tác tử phổ biến:
- WebArena / VisualWebArena
- WorkArena
- SWE-bench / InterCode

## Ranh giới tài nguyên và vòng đời

- Máy chủ phục vụ hoạt động cục bộ, tiêu thụ bộ nhớ tối thiểu (~50-100MB).
- Không yêu cầu quyền root hoặc Docker đối với các mô phỏng dựa trên HTTP thuần túy.
- Thời gian chờ mặc định cho mỗi yêu cầu là 30 giây để ngăn chặn tác tử bị treo vô hạn.
