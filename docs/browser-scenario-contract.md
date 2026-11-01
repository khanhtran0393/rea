# Hợp đồng Kịch bản Trình duyệt (Browser Scenario Contract)

Công cụ `capture_browser_scenario` chấp nhận lược đồ trung lập với nhà cung cấp `browserScenarioSchema` thông qua cả MCP và lệnh CLI `capture-browser-scenario INPUT_JSON`. Yêu cầu có tính chất khai báo: nó thừa nhận một tập từ vựng hành động cố định, các mục tiêu HTTP(S) rõ ràng, cài đặt trình duyệt tất định, giá trị khởi tạo lưu trữ rõ ràng và thời hạn kiểm tra liveness của nhà cung cấp. `INPUT_JSON` có thể là chuỗi JSON nội dòng hoặc đường dẫn tệp JSON.

Yêu cầu tối thiểu chỉ chứa cấu hình khởi chạy/kết nối trình duyệt, URL khởi đầu `start_url` và danh sách các bước hành động.

## Xác thực kịch bản trước khi chạy

Kiểm tra tính hợp lệ của kịch bản JSON trước khi kích hoạt trình duyệt:

```sh
rea validate-browser-scenario ./scenario.json
```

Lệnh này kiểm tra lược đồ cú pháp, các hành động hợp lệ, tính hợp lệ của bộ chọn DOM (CSS/XPath), và đảm bảo không có xung đột thời gian chờ.

## Ví dụ khởi chạy tối thiểu (Minimal launch example)

```json
{
  "browser": {
    "type": "chromium",
    "headless": true
  },
  "start_url": "https://example.com/login",
  "steps": [
    {
      "action": "navigate",
      "url": "https://example.com/login"
    },
    {
      "action": "wait_for_selector",
      "selector": "#username"
    }
  ]
}
```

## Ví dụ tương tác: Ghi lại kết quả hiển thị

```json
{
  "browser": {
    "type": "chromium",
    "viewport": { "width": 1280, "height": 800 }
  },
  "start_url": "https://example.com/dashboard",
  "steps": [
    {
      "action": "fill",
      "selector": "#username",
      "value": "auditor_user"
    },
    {
      "action": "fill",
      "selector": "#password",
      "value": "SecretPassword123"
    },
    {
      "action": "click",
      "selector": "button[type='submit']"
    },
    {
      "action": "wait_for_navigation"
    },
    {
      "action": "screenshot",
      "name": "dashboard_loaded"
    }
  ]
}
```

## Các bước yêu cầu (Request steps)

Các hành động được hỗ trợ trong danh sách `steps`:
- `navigate`: Chuyển hướng trình duyệt đến một URL cụ thể.
- `click`: Nhấp chuột vào một phần tử giao diện theo bộ chọn.
- `fill`: Điền chuỗi văn bản vào trường nhập liệu.
- `hover`: Di chuột qua một phần tử.
- `press_key`: Gửi phím nhấn bàn phím (ví dụ: `Enter`, `Tab`).
- `wait_for_selector`: Chờ một phần tử xuất hiện trong DOM.
- `wait_for_navigation`: Chờ hoàn thành điều hướng trang.
- `screenshot`: Chụp ảnh màn hình trạng thái trang hiện tại.
- `evaluate_script`: Thực thi một đoạn mã JavaScript nhỏ trong ngữ cảnh trang (bị giới hạn trong môi trường an toàn).

## Giá trị đã lưu (Saved values)

Kịch bản có thể lưu các giá trị trích xuất từ DOM để sử dụng trong các bước tiếp theo hoặc ghi vào Bằng chứng:

```json
{
  "action": "save_text",
  "selector": ".confirmation-token",
  "save_as": "token_value"
}
```

## Bằng chứng Mạng (Network evidence)

Trong suốt quá trình thực thi kịch bản, toàn bộ hoạt động mạng được thu thập:
- Mọi yêu cầu HTTP/HTTPS, phản hồi, tiêu đề và mã trạng thái.
- Định dạng HAR chuẩn được tích hợp trực tiếp vào tệp kết quả Evidence.

## Dấu vân tay giá trị lưu trữ (Storage value fingerprints)

Ghi nhận dấu vân tay mã hóa (SHA-256) của các khóa lưu trữ:
- Cookies của miền đích.
- Cặp khóa-giá trị trong `localStorage` và `sessionStorage`.
- Giúp phát hiện sự thay đổi trạng thái phiên mà không làm rò rỉ dữ liệu nhạy cảm trong log công khai.

## So sánh kịch bản (Scenario comparison)

So sánh hai lần chạy kịch bản để phân tích hồi quy hoặc kiểm tra sự phân kỳ:

```sh
rea compare-browser-scenarios ./run1.json ./run2.json --output ./diff.json
```

Báo cáo phân tích:
- Sự thay đổi về cấu trúc cây DOM tại từng bước.
- Độ lệch thời gian phản hồi mạng.
- Sự sai khác hình ảnh giữa các ảnh chụp màn hình tương ứng.

## Cung cấp tóm tắt và chế độ xem lưu giữ

Khi kết quả kịch bản quá lớn, REA cung cấp chế độ tóm tắt gọn gàng kèm theo ID Bằng chứng lưu giữ. Tác tử có thể truy vấn sâu hơn vào từng bước cụ thể bằng công cụ `inspect_analysis_view`.
