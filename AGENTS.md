# Hướng dẫn dành cho Tác tử AI (Agents Guide)

Tệp này cung cấp ngữ cảnh kiến trúc và các chỉ dẫn quan trọng nhất dành cho các Tác tử AI (Claude, Antigravity, Gemini, Codex, DeepSeek) khi làm việc trong kho lưu trữ REA.

## Kiến trúc Tổng quan

REA là một bộ công cụ phân tích kỹ thuật dịch ngược và quan sát thời gian chạy thống nhất, bao gồm:
1. **Core Engine (`src/core/`)**: Quản lý vòng đời Bằng chứng (Evidence), bộ nhớ đệm và các hợp đồng kiểu dữ liệu.
2. **Bộ phân tích Nhị phân (`src/native/`, `src/dotnet/`, `src/android/`)**: Tích hợp Ghidra, Hopper, IDA, JADX và các trình phân tích tĩnh định dạng PE/ELF/Mach-O.
3. **Bộ quan sát Web & Mạng (`src/web/`, `src/network/`)**: Quan sát trình duyệt qua CDP, ghi nhận kịch bản tương tác và phân tích tệp bắt gói tin (HAR/SAZ/PCAP).
4. **Môi trường Tác tử (`src/environments/`)**: Xây dựng các gói môi trường mô phỏng (AgentEnv bundles) cho phép tác tử AI tương tác và huấn luyện ngoại tuyến.
5. **Giao diện MCP & CLI (`src/mcp/`, `bin/`)**: Cung cấp giao thức Model Context Protocol và giao diện dòng lệnh cho người dùng và các tác tử AI.

## Các Quy tắc Vàng cho Tác tử

- **Bảo toàn Tính nguyên vẹn**: Không bao giờ sửa đổi tệp nhị phân mục tiêu trừ khi được yêu cầu rõ ràng.
- **Sử dụng Bằng chứng Lưu giữ**: Tận dụng `evidence_id` để tránh tràn cửa sổ ngữ cảnh của mô hình ngôn ngữ.
- **Xác thực Thực tế**: Luôn chạy lệnh kiểm tra thực tế trên terminal để xác minh kết quả thay vì suy diễn.
