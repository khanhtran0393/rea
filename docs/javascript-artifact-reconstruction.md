# Tái cấu trúc Artifact JavaScript (JavaScript Artifact Reconstruction)

Tài liệu này mô tả quy trình phân tích và tái cấu trúc mã nguồn từ các gói JavaScript đã được làm mờ (obfuscated), thu nhỏ (minified) hoặc đóng gói thành tệp lưu trữ (ASAR, SEA, Bun standalone).

## Quy trình Công khai (Public Workflow)

Quy trình tái cấu trúc được tổ chức theo các bước chuẩn mực:
1. **Trích xuất Artifact**: Giải nén gói lưu trữ (ví dụ: `app.asar`, tệp nhị phân SEA) để lấy các tệp kịch bản thô.
2. **Khôi phục Sourcemap**: Tự động phát hiện sourcemap nội dòng hoặc tệp `.map` ngoài, tái tạo lại cây thư mục gốc.
3. **Định dạng & Khử làm mờ**: Sử dụng AST deobfuscator để chuẩn hóa tên biến, cấu trúc luồng điều khiển và hằng số chuỗi.
4. **Trích xuất Đồ thị Ứng dụng**: Xây dựng lại danh mục module và các phụ thuộc `package.json`.

```sh
rea reconstruct-js ./dist/bundle.js --output-dir ./reconstructed/ --json
```

## Lựa chọn Nguồn trước khi Phân tích (Select sources before analysis)

Đối với các ứng dụng lớn có hàng trăm tệp, bạn có thể chỉ định các tệp quan trọng:
- Sử dụng bộ lọc `--include-glob "src/**/*.js"` hoặc `--exclude-vendor` để tập trung vào mã nguồn ứng dụng độc quyền.
- Bỏ qua các thư viện bên thứ ba nổi tiếng (React, Lodash, jQuery) để tiết kiệm thời gian và tài nguyên xử lý.

## Đọc kết quả Phân tích (Read the analysis)

Báo cáo phân tích Evidence cung cấp:
- Danh sách các tệp đã được khôi phục kèm theo đường dẫn đề xuất.
- Bảng ánh xạ tên hàm/biến (symbol mapping table).
- Đánh giá mức độ hoàn thiện của từng tệp tái cấu trúc.

## Tính toàn vẹn của ASAR (ASAR Integrity)

Đối với các ứng dụng Electron:
- Kiểm tra tính toàn vẹn của tiêu đề ASAR và mã băm SHA-256 của từng tệp con.
- Xác định cờ bảo vệ `app.asar.integrity` và Electron fuses liên quan đến việc xác thực mã nguồn.

```sh
rea inspect-asar ./resources/app.asar --check-integrity --json
```

## Kết quả Phân tích Lớn (Large results)

Đối với các dự án lớn, REA hỗ trợ lưu trữ kết quả theo phân đoạn (chunked output) hoặc lưu trực tiếp vào cơ sở dữ liệu SQLite trung gian để tránh tràn bộ nhớ RAM.

## Khôi phục Phân tích Một phần (Recover partial analysis)

Nếu quá trình phân tích bị gián đoạn hoặc gặp lỗi trên một số module bị bảo vệ nghiêm ngặt:
- REA vẫn bảo toàn kết quả của tất cả các module đã xử lý thành công.
- Tác tử có thể tiếp tục phân tích từ điểm dừng cuối cùng bằng cờ `--resume`.

## Những gì được Tái cấu trúc (What is reconstructed)

- **Cây thư mục**: Cấu trúc thư mục mã nguồn ban đầu.
- **Mã nguồn định dạng chuẩn**: Cú pháp ES6+/TypeScript có thụt lề và định dạng dễ đọc.
- **Hệ thống module**: Khôi phục các câu lệnh `import` và `export` chuẩn thay cho các hàm đóng gói phức tạp của bundler.
- **Tài sản đính kèm (Assets)**: Trích xuất các tệp hình ảnh, CSS, JSON và tài nguyên tĩnh đi kèm.

## Thẩm quyền và Những điều Chưa biết (Authority and unknowns)

REA phân định rõ ràng giữa:
- **Dữ kiện đã xác minh**: Tên biến/hàm được khôi phục trực tiếp từ sourcemap hoặc debug symbols.
- **Dữ kiện suy luận**: Tên hàm được tác tử AI đề xuất dựa trên phân tích ngữ cảnh (được đánh dấu rõ ràng với cờ `inferred: true`).

## Ranh giới An toàn (Safety Boundary)

REA không bao giờ thực thi mã JavaScript không đáng tin cậy trong môi trường phân tích tĩnh. Toàn bộ quá trình phân tích cú pháp được thực hiện an toàn trên cây cú pháp trừu tượng (AST) thông qua bộ phân tích Babel/Acorn được cô lập.

## Giới hạn và Độ bao phủ (Bounds and Coverage)

- **Nhập JSON cục bộ lớn**: Áp dụng giới hạn nạp luồng đối với các tệp JSON tài nguyên vượt quá 100MB.
- **Thời hạn phân tích**: Đặt giới hạn thời gian tối đa cho mỗi module phức tạp để ngăn ngừa vòng lặp vô hạn trong các thuật toán khử làm mờ.

## Ranh giới Xác minh (Verification boundary)

Mã nguồn tái cấu trúc được xác minh cú pháp tự động thông qua ESLint và TypeScript compiler để đảm bảo không chứa lỗi cú pháp trước khi bàn giao cho người dùng.
