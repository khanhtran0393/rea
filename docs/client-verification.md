# Xác minh Ứng dụng Khách và Tác tử (Client and Agent Verification)

Tài liệu này mô tả quy trình kiểm thử và xác minh tính tương thích giữa máy chủ MCP REA và các ứng dụng khách, tác tử AI tự trị, cùng các môi trường triển khai thực tế.

## Xác minh Môi trường Khởi chạy (Launch environment verification)

Trước khi kích hoạt các tác tử tự trị, môi trường máy khách phải được xác thực để đảm bảo:
- Khả năng tạo tiến trình con (`child_process.spawn`).
- Kết nối thông suốt qua đường ống `stdio` hoặc giao thức HTTP `sse`.
- Quyền truy cập tệp nhị phân mục tiêu và các thư mục làm việc tạm thời.

Lệnh kiểm tra tính sẵn sàng của môi trường:

```sh
rea doctor --client-check --json
```

## Giới hạn Bộ hiển thị Chế độ Mã nguồn của Pi (Published Pi code-mode renderer limit)

Ứng dụng khách Pi áp dụng giới hạn kích thước khối đầu ra hiển thị. Khi kết quả phân tích vượt quá ngưỡng này, REA tự động kích hoạt cơ chế phân trang và lưu giữ Bằng chứng (Retained Evidence), trả về bản tóm tắt gọn gàng kèm theo ID Bằng chứng để tác tử truy vấn chi tiết từng phần.

## Bằng chứng Đầu-cuối, Tích hợp và Vàng (End-to-end, Integration and Golden Evidence)

Bộ kiểm thử của REA bao gồm:
- **E2E Tests**: Khởi chạy một tiến trình máy khách thật, gửi các yêu cầu MCP đại diện, và kiểm tra tính toàn vẹn của phản hồi.
- **Integration Tests**: Kiểm tra sự phối hợp giữa REA và các công cụ dịch ngược bên ngoài (Ghidra, JADX, CDP).
- **Golden Evidence**: So sánh kết quả phân tích với bộ bằng chứng chuẩn được kiểm duyệt trước (golden records) để ngăn chặn hiện tượng hồi quy dữ liệu hoặc suy giảm độ chính xác.

## Hồ sơ Đánh giá và Tuân thủ của Tác tử (Agent evaluation and conformance records)

REA theo dõi các chỉ số thực hiện tác vụ của tác tử AI:
- **Tỷ lệ thành công của tác vụ**: Khả năng tác tử sử dụng đúng công cụ REA để giải quyết bài toán dịch ngược hoặc trích xuất thông tin.
- **Hiệu quả ngữ cảnh (Context efficiency)**: Lượng token tiêu thụ để đạt được câu trả lời chính xác.
- **Tỷ lệ lỗi công cụ (Tool call error rate)**: Số lần gọi sai tham số hoặc vượt quá thời hạn cho phép.

Báo cáo tuân thủ được lưu trữ và cập nhật trong các bài kiểm tra tích hợp liên tục (CI).
