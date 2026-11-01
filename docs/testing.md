# Kiểm thử REA (Testing REA)

Tài liệu này mô tả toàn diện hệ thống kiểm thử, bộ xác minh tự động và các quy trình đảm bảo chất lượng (QA) của kho lưu trữ REA.

## Lệnh Dành cho Nhà phát triển (Developer Commands)

```sh
# Chạy toàn bộ các bài kiểm tra đơn vị (unit tests)
npm test

# Chạy kiểm tra định dạng và quy tắc linter
npm run lint

# Chạy kiểm tra kiểu tĩnh TypeScript
npm run typecheck

# Chạy kiểm thử với báo cáo độ bao phủ mã
npm run test:coverage
```

## Lựa chọn Quy trình Xác minh cho Thay đổi

Khi bạn thực hiện một thay đổi mã nguồn, hãy chọn làn kiểm thử (test lane) phù hợp:
- **Thay đổi Core / CLI**: Chạy `npm run test:core` và `npm run test:cli`.
- **Thay đổi Bộ phân tích Nhị phân**: Chạy các bài kiểm thử tương ứng (`npm run test:native`, `npm run test:dotnet`).
- **Thay đổi Tác tử / MCP**: Chạy `npm run test:mcp` và kiểm tra hợp đồng lược đồ.

### Xác minh Quy trình Dựa trên Nhà cung cấp

Các bài kiểm thử yêu cầu công cụ bên ngoài (Ghidra, IDA, JADX) được thiết kế để tự động bỏ qua (skip) một cách an toàn nếu công cụ đó không có mặt trên máy cục bộ, nhưng sẽ chạy bắt buộc trong môi trường CI chính thức.

## Thiết kế Bài kiểm thử và Cắt tỉa (Test design and pruning)

REA tuân thủ nguyên tắc kiểm thử nghiêm ngặt:
- Mỗi bài kiểm thử phải độc lập, không phụ thuộc vào thứ tự chạy hoặc trạng thái lưu trữ chung.
- Không sử dụng dữ liệu giả lập (mocking) hời hợt đối với các bộ phân tích nhị phân cốt lõi; ưu tiên kiểm thử trên các tệp mẫu nhị phân thực tế nhỏ (golden fixtures).

### Hồi quy Ranh giới (Boundary regressions)

Mỗi lỗi được phát hiện phải có một bài kiểm thử hồi quy tương ứng để đảm bảo lỗi không bao giờ xuất hiện lại trong tương lai.

## Hồi quy Vòng đời và Độ sâu Hành vi

Kiểm tra tính đúng đắn của việc giải phóng tài nguyên:
- Các tiến trình con (Ghidra headless, CDP browser) phải được dọn dẹp sạch sẽ khi lệnh kết thúc.
- Không làm rò rỉ các tệp tạm thời trong thư mục hệ thống.

## Bằng chứng Đầu-cuối, Tích hợp và Vàng (Golden Evidence)

Các kết quả phân tích chuẩn đối với các tệp nhị phân tham chiếu được lưu trữ trong thư mục `tests/fixtures/golden/`. Mọi thay đổi làm thay đổi kết quả đầu ra Bằng chứng đều phải được kiểm duyệt thủ công và cập nhật có chủ đích.

## Độ bao phủ và Thời gian chạy

Mục tiêu độ bao phủ mã nguồn của REA là trên 85% cho các module cốt lõi. Thời gian chạy của bộ kiểm tra đơn vị cục bộ được tối ưu dưới 30 giây.

## Các Làn Xác minh Chuỗi công cụ Thực tế (Real-toolchain verification lanes)

REA duy trì các làn kiểm thử chuyên biệt:
- **Phân tích Android APK**: Kiểm tra JADX trên các tệp APK mẫu thực tế.
- **Giải mã Tài nguyên Apktool**: Kiểm tra trích xuất tài nguyên XML nhị phân và ảnh chín đoạn.
- **Nhận diện Bản dựng Flutter**: Nhận diện cấu trúc engine Flutter và tệp snapshot `libapp.so`.
- **Thu thập Thiết bị ADB**: Kiểm tra giao tiếp với máy ảo hoặc thiết bị Android thật.
- **Phân tích Ghidra NativeAOT**: Xác minh khả năng khôi phục metadata trên các bản dựng .NET AOT.
- **Bộ điều hợp IDA MCP**: Kiểm tra tích hợp RPC với IDA Pro.
- **Làn Kuna**: Kiểm tra bộ giải mã và trích xuất chuyên dụng.
- **Phân tích DOS Ghidra**: Kiểm tra các tệp nhị phân 16-bit MZ / NE cổ điển.
- **Lưu trữ Apple Interface Builder**: Kiểm tra giải mã NIB/XIB và Storyboard.
- **Phân tích Bắt gói tin Mạng (PCAP)**: Kiểm tra phân tích giao thức TCP/UDP, TLS handshake và luồng HTTP/2.
- **Khôi phục Mã nguồn JavaScript**: Xác minh tính chính xác của việc tái tạo AST từ các gói mã thu nhỏ.
- **Công cụ Frida**: Xác minh việc tiêm mã và chặn hàm động thời gian chạy.
- **Artifacts WASM**: Kiểm tra giải mã tệp bytecode WebAssembly.
