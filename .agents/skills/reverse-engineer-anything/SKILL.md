---
name: reverse-engineer-anything
description: Dịch ngược (Reverse engineer) các mục tiêu native, managed, Electron/JavaScript, packaged, firmware và trình duyệt với REA, đồng thời xây dựng môi trường tác tử (AgentEnv bundles) của ứng dụng web từ phiên bản trực tiếp hoặc bản ghi lưu lượng. Sử dụng bằng chứng từ bản dựng phát hành (shipped-artifact) hoặc bằng chứng runtime được yêu cầu để giải thích tính năng, so sánh phiên bản, dịch ngược mã hoặc định hướng tái cấu trúc mã nguồn. Bỏ qua REA khi phân tích kho mã nguồn thông thường.
metadata:
  version: "58"
---

# REA (Reverse Engineer Anything)

Sử dụng REA khi mục tiêu phân tích là một tệp nhị phân đã phát hành (shipped artifact), tệp đóng gói, firmware, trang web hoặc phiên làm việc đã ghi lại mà bạn cần trích xuất bằng chứng kỹ thuật chính xác.

## Khi nào nên Sử dụng REA

1. **Phân tích Nhị phân Native**: Tệp thực thi PE (.exe, .dll, .sys), ELF (.so, binary), Mach-O (macOS, iOS), DOS, firmware nhúng (UF2, DfuSe, FIT, uImage, ESP).
2. **Mã nguồn Quản lý**: Ứng dụng .NET (CIL, NativeAOT, SingleFileHost), Java/Android APK, Unity IL2CPP / Mono.
3. **Ứng dụng Web và Desktop JavaScript**: Các gói Webpack/Vite/Rollup thu nhỏ, Electron ASAR, Node.js SEA, Bun standalone.
4. **Môi trường Tác tử**: Xây dựng gói phát lại ngoại tuyến từ các bản ghi lưu lượng mạng (HAR, SAZ) hoặc kịch bản duyệt web.

## Quy trình Làm việc Khuyến nghị

1. **Kiểm tra Tổng quan Artifact**:
   ```sh
   rea inspect-artifact ./target --json
   ```
2. **Khám phá Chi tiết theo Nền tảng**:
   - Đối với mã Native: `rea inspect-native ./target --functions --json`
   - Đối với .NET: `rea inspect-assembly ./target.dll --json`
   - Đối với Android APK: `rea inspect-apk ./target.apk --json`
   - Đối với Lưu lượng mạng: `rea inspect-capture ./traffic.har --json`
3. **Dịch ngược và Trích xuất**:
   - Khôi phục mã giả C qua Ghidra: `rea inspect-native ./target --function main --decompile --json`
   - Trích xuất tài nguyên và cấu trúc mã nguồn tái cấu trúc.
