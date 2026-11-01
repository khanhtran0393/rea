# Hướng dẫn Đóng góp (Contributing Guide)

Cảm ơn bạn đã quan tâm đến việc cải thiện REA! Chúng tôi chào đón các đóng góp từ cộng đồng nghiên cứu an ninh và phát triển phần mềm.

## Thiết lập Môi trường Phát triển

1. Clone kho lưu trữ:
   ```sh
   git clone https://github.com/khanhtran0393/rea.git
   cd rea
   ```
2. Cài đặt các phụ thuộc:
   ```sh
   npm install
   ```
3. Xây dựng dự án:
   ```sh
   npm run build
   ```

## Quy trình Kiểm thử Bắt buộc

Trước khi tạo Pull Request, hãy đảm bảo tất cả các bài kiểm tra đều vượt qua:
```sh
npm test
npm run lint
npm run typecheck
```

## Quy tắc Viết mã và Đảm bảo Chất lượng

- **Không sử dụng mock ảo**: Ưu tiên kiểm thử trên các tệp mẫu nhị phân thực tế nhỏ (golden fixtures).
- **Tuân thủ hợp đồng Bằng chứng (Evidence Contracts)**: Đảm bảo mọi công cụ trả về định dạng JSON có cấu trúc nhất quán.
- **Tương thích đa nền tảng**: Mã nguồn phải chạy trơn tru trên Windows, Linux và macOS.

## Tạo Pull Request (PR)

- Đặt tên nhánh mô tả rõ ràng (ví dụ: `fix/pe-load-config`, `feat/wasm-parser`).
- Viết thông điệp commit rõ ràng theo chuẩn Conventional Commits (`feat:`, `fix:`, `docs:`).
- Cập nhật tài liệu tương ứng trong thư mục `docs/`.
