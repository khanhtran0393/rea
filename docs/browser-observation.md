# Quan sát Trang web bằng Chrome DevTools Protocol (CDP)

Mã nguồn được thu thập từ Debugger có thể được xuất cho phân tích JavaScript tĩnh hiện có của REA bằng `export_web_scripts`. Xem [tài liệu kịch bản trang web đã thu thập](website-script-export.md) để biết các tùy chọn thu thập, ánh xạ nguồn (source maps), và các giới hạn phân giải cục bộ/trình duyệt. Sử dụng [mối quan hệ module đã thu thập](website-module-trace.md) để giải quyết các import native của một kịch bản đã xuất theo ngữ cảnh URL/import-map rõ ràng.

REA có thể kết nối với một trình duyệt thuộc họ Chrome do người dùng sở hữu thông qua giao thức CDP để ghi lại hoạt động trang web, chụp ảnh màn hình, lưu vết mạng và trích xuất DOM.

## Các bề mặt được hỗ trợ (Shipped surfaces)

- **Google Chrome, Chromium, Brave, Microsoft Edge**.
- Trình duyệt khởi chạy cục bộ hoặc phiên chạy ngầm không đầu (headless).
- Trình duyệt có cấu hình cổng gỡ lỗi từ xa (`--remote-debugging-port`).

## Ngân sách bộ nhớ so sánh ảnh chụp màn hình

Khi thực hiện so sánh trực quan giữa các bước duyệt web:
- Ảnh chụp màn hình được nén và lưu trữ hiệu quả trong bộ đệm bằng chứng.
- Hỗ trợ so sánh ngưỡng khác biệt pixel (SSIM / Pixel-by-pixel diffing).
- Bộ nhớ dành riêng cho so sánh hình ảnh được giới hạn tối đa 512MB để tránh ảnh hưởng đến tiến trình chính.

## Quyền hạn của nhà cung cấp (Provider Authorities)

REA hoạt động với vai trò là một client gỡ lỗi CDP thụ động:
- Nó không tự ý tiêm mã độc hại hoặc thay đổi logic trang web trừ khi có chỉ thị trong kịch bản.
- Quyền truy cập cookie, mã thông báo phiên và bộ nhớ cục bộ tuân theo chính sách cùng nguồn gốc (SOP) của trình duyệt.

## Khởi động một trình duyệt

Khởi động trình duyệt Chromium với cổng gỡ lỗi được kích hoạt:

```sh
google-chrome --remote-debugging-port=9222 --user-data-dir=/tmp/chrome-debug
```

Sau đó kết nối REA với trình duyệt:

```sh
rea observe-browser --cdp-url http://127.0.0.1:9222 --url https://target.example.com --json
```

### Các trình duyệt không có khám phá HTTP (Browsers without HTTP discovery)

Một số môi trường bảo mật vô hiệu hóa điểm cuối `/json/version`. Trong trường hợp này, bạn có thể truyền trực tiếp URL WebSocket:

```sh
rea observe-browser --cdp-ws-url ws://127.0.0.1:9222/devtools/browser/<guid> --json
```

## Ranh giới yêu cầu (Request boundary)

Mỗi phiên quan sát CDP tuân theo ranh giới phiên rõ ràng:
- Lắng nghe sự kiện `Network.requestWillBeSent`, `Network.responseReceived`, `Page.domContentEventFired`.
- Thu thập tất cả các tài nguyên liên quan đến trang mục tiêu cho đến khi sự kiện tải trang hoàn tất hoặc hết thời gian chờ (deadline).

## Quy trình làm việc trên CLI

```sh
# Thu thập ảnh chụp nhanh phiên trang web hoàn chỉnh
rea observe-browser --url https://example.com --screenshot ./screenshot.png --har ./network.har --json

# Chụp cây DOM và các phần tử tương tác
rea inspect-web-page --url https://example.com --dom-dump ./dom.json --json
```

### Điều kiện tiên quyết để khám phá WebMCP

Để tự động phát hiện các endpoint WebMCP:
- Đảm bảo trang web đích cung cấp metadata hoặc tài nguyên khai báo công khai.
- Cổng kết nối CDP phải mở và có thể truy cập được từ tiến trình REA.

## Đầu vào MCP

Đối với máy chủ MCP, công cụ `observe_website` chấp nhận các tham số:
- `url`: Địa chỉ trang web cần quan sát.
- `capture_network`: Boolean (có ghi nhận HAR hay không).
- `capture_dom`: Boolean (có trích xuất cây DOM hay không).
- `screenshot`: Boolean (có chụp ảnh màn hình hay không).
- `wait_for_selector`: Chuỗi CSS selector chờ xuất hiện trước khi kết thúc thu thập.

## Giảm thiểu dữ liệu (Data Minimization)

Để bảo vệ quyền riêng tư và dữ liệu nhạy cảm:
- Các trường mật khẩu (`input[type="password"]`) tự động được làm mờ/loại trừ khỏi bản ghi DOM.
- Các tiêu đề xác thực nhạy cảm (`Authorization`, `Cookie`) có thể được băm hoặc ẩn danh hóa thông qua cờ cấu hình `--sanitize-secrets`.

## Tính đầy đủ và Giới hạn

- **Hỗ trợ đầy đủ**: Các ứng dụng Single Page Apps (SPA) xây dựng trên React, Vue, Angular, Svelte.
- **Giới hạn**: Không thể vượt qua CAPTCHA hoặc xác thực 2 lớp sinh trắc học nếu không có sự can thiệp thủ công của người vận hành; các kết nối WebRTC ngang hàng phức tạp chỉ được ghi lại ở mức tín hiệu kết nối.

## Phi mục tiêu và Mô hình đe dọa

REA là công cụ hỗ trợ phân tích và kỹ thuật dịch ngược hợp pháp cho các hệ thống thuộc sở hữu hoặc được ủy quyền. Nó không được thiết kế để vượt qua các hệ thống kiểm soát truy cập bất hợp pháp hoặc tấn công từ chối dịch vụ.
