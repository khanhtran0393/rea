# Quy trình Ứng dụng JavaScript Đa tầng (Cross-layer JavaScript Application Workflows)

REA cung cấp các quy trình làm việc đa tầng để phân tích các ứng dụng JavaScript/TypeScript quy mô lớn, bao gồm các ứng dụng web SPA, ứng dụng Node.js phía máy chủ, và các gói Electron/Tauri desktop. Các quy trình này kết nối thông tin giữa mã đóng gói (bundled/minified), đồ thị module, khai báo native, và hành vi thời gian chạy.

## Dữ kiện Module được Lưu giữ (Retained Module Facts)

Khi phân tích một gói mã JavaScript, REA trích xuất và lưu giữ:
- **Đồ thị phụ thuộc module**: Các quan hệ `import`, `export`, `require`, và nạp động `import()`.
- **Định danh module**: Ánh xạ giữa ID module số học của Webpack/Vite/Rollup với đường dẫn tương đối gốc nếu có sourcemap.
- **Biến và hàm xuất khẩu**: Danh sách các API công khai của từng module.
- **Phân đoạn mã (Chunk splits)**: Các điểm chia tách gói mã tải lười (lazy loading chunks).

## Dữ kiện Tệp và Native được Lưu giữ

Đối với các ứng dụng desktop hoặc môi trường lai:
- Phân tích các liên kết tới mô-đun Node C++ Addons (`.node`).
- Khám phá các hàm FFI (`node-ffi-napi`, `bun:ffi`, `koffi`).
- Ánh xạ các kênh liên lạc IPC giữa tiến trình chính (Main process) và tiến trình hiển thị (Renderer) trong Electron.

## Truy vết Tính năng (Feature Tracing)

Truy vết từ giao diện người dùng hoặc điểm cuối API xuống mã nguồn cốt lõi:
- Xác định chuỗi gọi hàm từ nút bấm UI đến hàm xử lý logic nghiệp vụ.
- Nhận diện các mẫu thiết kế phổ biến: Redux actions/reducers, Zustand stores, Vuex/Pinia, React Context.

## Truy vết Mối quan hệ Ngữ nghĩa (Semantic Relation Tracing)

Phân tích quan hệ dữ liệu ngữ nghĩa:
- Luồng dữ liệu giữa các thành phần giao diện và các dịch vụ mạng.
- Ánh xạ giữa các mô hình dữ liệu (TypeScript interfaces / Zod schemas) và các endpoint REST/GraphQL tương ứng.

## So sánh Phiên bản (Version Comparison)

So sánh hai bản dựng của cùng một ứng dụng:

```sh
rea compare-js-apps ./v1.0/bundle.js ./v2.0/bundle.js --json
```

Báo cáo phân tích:
- Các module mới được thêm vào hoặc bị loại bỏ.
- Thay đổi chữ ký hàm của các API công khai.
- Biến động về kích thước mã nguồn và phụ thuộc bên thứ ba.

## So sánh Mã nguồn Lịch sử với Gói Mã (Historical Source-to-Bundle Comparison)

Khi bạn có mã nguồn của phiên bản cũ và tệp đã đóng gói của phiên bản mới:
- REA tự động so khớp các hàm tương đồng qua cấu trúc AST và dấu vân tay ngữ nghĩa.
- Đánh dấu các khối mã không thay đổi và các khối mã mới được cập nhật hoặc chỉnh sửa.

## So sánh Hình dạng Trả về của Xuất khẩu (Export return-shape comparison)

Kiểm tra tính tương thích ngược của các module thư viện:
- Phát hiện các thay đổi phá vỡ (breaking changes) trong cấu trúc đối tượng trả về.
- Xác định các trường dữ liệu bị đổi tên, thêm mới hoặc xóa bỏ.

## CLI và Xác minh

```sh
# Phân tích luồng công việc ứng dụng JavaScript
rea analyze-js-app ./dist/app.js --entrypoint ./main.js --json

# Kiểm tra tính toàn vẹn của đồ thị module
rea verify-module-graph ./dist/app.js --json
```

## Bằng chứng Thời gian chạy (Runtime Evidence)

Tích hợp dữ liệu quan sát thời gian chạy từ CDP (Chrome DevTools Protocol) để:
- Xác nhận các nhánh mã thực sự được thực thi trong quá trình tương tác.
- Lọc bỏ mã chết (dead code) không bao giờ được kích hoạt trong kịch bản thực tế.
