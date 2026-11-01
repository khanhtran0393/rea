# Cài đặt và Thiết lập (Installation and Setup)

REA được đóng gói dưới dạng gói npm `@morluto/rea`, cung cấp cả giao diện dòng lệnh CLI và máy chủ Model Context Protocol (MCP) cho các tác tử AI.

## Bắt đầu thiết lập

Bạn có thể cài đặt toàn cục bằng npm:

```sh
npm install -g @morluto/rea
```

Hoặc chạy tức thì không cần cài đặt qua `npx`:

```sh
npx @morluto/rea --help
```

Sau khi cài đặt, chạy trình hướng dẫn thiết lập tự động:

```sh
rea setup
```

Lệnh này sẽ kiểm tra môi trường, phát hiện các công cụ đã cài đặt (Java, Ghidra, JADX, IDA), và tùy chọn đăng ký máy chủ MCP vào các ứng dụng khách được hỗ trợ.

## Xem xét các thay đổi thiết lập

Trình hướng dẫn thiết lập sẽ hiển thị bản tóm tắt các tệp cấu hình mà nó dự định sửa đổi trước khi thực hiện:
- Đích tệp cấu hình Claude Desktop.
- Đích cấu hình VS Code / Continue / Cline.
- Cấu hình biến môi trường hoặc liên kết tượng trưng (symlinks).

### Đích đến của Skill (Skill destinations)

REA có thể tự động cài đặt các định nghĩa kỹ năng (skills) vào:
- `~/.gemini/config/skills/rea/` (cho Antigravity / Gemini)
- `~/.claude/skills/rea/` (cho Claude Code)

## Giới hạn REA trong một dự án hoặc hồ sơ tác tử

Nếu bạn không muốn cài đặt toàn cục, bạn có thể thiết lập REA cục bộ trong một thư mục dự án:

```sh
npm install --save-dev @morluto/rea
npx rea init-project
```

## Kiểm tra tính sẵn sàng cho tác vụ của bạn

Chạy lệnh chẩn đoán tích hợp:

```sh
rea doctor
```

Lệnh này đánh giá:
- Phiên bản Node.js (yêu cầu Node 18+).
- Trạng thái Java Runtime (JRE/JDK 11+ cho Android và Ghidra).
- Quyền truy cập các công cụ dịch ngược (Ghidra, Hopper, IDA).
- Tính khả dụng của trình duyệt Chromium cho các tác vụ quan sát trang web.

### Điều kiện tiên quyết đối với tiến trình sở hữu trên Linux

Trên Linux, một số tác vụ quan sát tiến trình và chụp lưu lượng mạng yêu cầu quyền `ptrace` hoặc quyền `CAP_NET_ADMIN`/`CAP_NET_RAW`.

## Chẩn đoán, Cập nhật và Gỡ bỏ

```sh
# Chẩn đoán lỗi
rea doctor --verbose

# Cập nhật lên bản mới nhất
npm update -g @morluto/rea

# Gỡ bỏ cài đặt
npm uninstall -g @morluto/rea
```

### Xử lý thiếu phụ thuộc trong cài đặt npx

Nếu chạy qua `npx` gặp sự cố về bộ nhớ cache hoặc phụ thuộc bị thiếu, hãy xóa bộ nhớ cache npx:

```sh
npx --clear-cache
npx @morluto/rea@latest doctor
```

### Cập nhật bản cài đặt sở hữu

Nếu bạn làm việc trực tiếp từ kho lưu trữ mã nguồn Git:

```sh
git pull origin main
npm install
npm run build
```

## Gói phát hành và Nhánh Main

- **Bản phát hành npm**: Ổn định, được kiểm thử đầy đủ, phù hợp cho môi trường sản xuất.
- **Nhánh `main` trên GitHub**: Chứa các tính năng thử nghiệm mới nhất, các bộ phân tích tệp nhị phân đang phát triển và các bản sửa lỗi sớm.

## Cài đặt chỉ dành cho Kỹ năng (Skill-only installation)

Nếu bạn chỉ muốn các hướng dẫn tác tử và quy tắc mẫu mà không cần chạy máy chủ MCP nhị phân:

```sh
rea install-skill --dest ~/.gemini/config/skills/
```

## Các Tác tử được Hỗ trợ

REA hỗ trợ đầy đủ các tác tử AI: Claude, Antigravity, Gemini, DeepSeek, Kimi, Qwen, ChatGPT/Codex.

### DeepSeek Harness & Cài đặt máy khách khác

Xem [cấu hình máy khách](client-configuration.md) để biết chi tiết về cài đặt giới hạn lược đồ thu gọn (`--compact-schema`) khi làm việc với các mô hình có cửa sổ ngữ cảnh hẹp.

## Hopper

### Đường dẫn trình khởi chạy và xử lý sự cố

REA có thể giao tiếp với Hopper Disassembler trên macOS thông qua tập lệnh gỡ lỗi hoặc CLI của Hopper. Cấu hình đường dẫn:

```sh
export HOPPER_PATH="/Applications/Hopper Disassembler v4.app/Contents/MacOS/hopper"
```

### Hopper trong CI

Để chạy kiểm thử trong môi trường tự động hóa CI, đảm bảo giấy phép Hopper không cần tương tác giao diện đồ họa.

## Ghidra

### Trình dịch ngược native macOS bị thiếu

Nếu cài đặt Ghidra gặp lỗi thiếu trình dịch ngược nhị phân (`sleigh` hoặc `decompile`), hãy đảm bảo bạn đã cài đặt bộ phát triển dòng lệnh Xcode (`xcode-select --install`).

### Trình dịch ngược macOS arm64 (Apple Silicon)

Ghidra yêu cầu các tệp nhị phân hỗ trợ kiến trúc `osx_arm64`. Đảm bảo phiên bản Ghidra 10.2+ được sử dụng.

### Sử dụng lại dự án Ghidra hiện có

REA có thể mở và sử dụng lại một dự án Ghidra đã phân tích trước đó để tiết kiệm thời gian:

```sh
rea inspect-native ./binary --ghidra-project-dir ./ghidra_projects --ghidra-project-name MyProject --json
```

### Ngôn ngữ nhập và phân tích hạt giống của Ghidra

Cho phép chỉ định ID ngôn ngữ hoặc kiến trúc bộ xử lý nếu Ghidra không thể tự động phát hiện chính xác:

```sh
rea inspect-native ./raw_blob.bin --ghidra-language "ARM:LE:32:v7" --json
```

### Điều khiển bộ nhớ Heap và CPU của Ghidra

Điều chỉnh tài nguyên để phân tích các tệp nhị phân khổng lồ:

```sh
export GHIDRA_MAX_MEM="8G"
export GHIDRA_CPU_CORES="4"
```

### Chẩn đoán và Xác minh Ghidra

Chạy kiểm tra tình trạng kết nối Ghidra:

```sh
rea doctor --check-ghidra
```

## Plugin Tác tử và Chợ Ứng dụng (Agent plugins and marketplaces)

REA có thể được phát hiện và kích hoạt từ danh mục Agent Marketplace trong các IDE tương thích.

## Sổ đăng ký MCP (MCP Registry)

REA được đăng ký chính thức trên MCP Registry, cho phép cài đặt với một cú nhấp chuột trong các ứng dụng máy khách hỗ trợ tự động khám phá.

## Trình chạy gói và Cài đặt toàn cục

### Lỗi npm workspace trước khi REA khởi động

Nếu gặp sự cố xung đột workspace khi cài đặt cục bộ, hãy sử dụng cờ `--ignore-workspace-root-check` hoặc cài đặt toàn cục độc lập.

### Trình bọc cài đặt toàn cục (Global installation wrapper)

Gói cung cấp tệp nhị phân thực thi `rea` liên kết tới tệp entry point chính trong `bin/rea.js`.
