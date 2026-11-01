---
layout: home

hero:
  name: REA
  text: Kỹ Thuật Dịch Ngược Cho Mọi Thứ
  tagline: Thấu hiểu tệp nhị phân, ứng dụng và hành vi thời gian chạy chỉ với một CLI và MCP server.
  actions:
    - theme: brand
      text: Bắt đầu ngay
      link: /installation
    - theme: alt
      text: Xem trên GitHub
      link: https://github.com/khanhtran0393/rea

features:
  - title: Điều tra cùng tác tử của bạn
    details: Kết nối REA với AI agent lập trình của bạn, mô tả ứng dụng hoặc tính năng bạn muốn tìm hiểu và lần theo bằng chứng (Evidence).
    link: /mcp-prompts
  - title: Kiểm tra đa định dạng
    details: Khám phá tệp nhị phân native, ứng dụng JavaScript và Electron, assembly .NET, gói Android APK, firmware và website.
    link: /installation
  - title: Lưu giữ bằng chứng (Evidence)
    details: Phân tích chạy hoàn toàn cục bộ và trả về các quan sát, giới hạn và điểm chưa xác định qua CLI và giao thức MCP.
    link: /mcp-contracts
---

## Bắt đầu chỉ với một lệnh

```bash
npx rea-agents setup
```

Chọn tác tử của bạn, xem trước các thay đổi dự kiến và phê duyệt thiết lập. Việc phân tích tệp nhị phân native có thể tận dụng các bản cài đặt Hopper, Ghidra, hoặc IDA sẵn có; phân tích tĩnh JavaScript không cần bất kỳ engine native nào.

Để lấy kết quả phân tích đầu tiên từ một ứng dụng JavaScript hoặc gói ASAR đã giải nén:

```bash
npx -y rea-agents@latest analyze-javascript-application /duong/dan/tuyet/doi/toi/app --json
```

Xem [cài đặt và thiết lập](./installation.md) để biết các yêu cầu và cấu hình chi tiết.

Để theo dõi can thiệp thời gian chạy (runtime instrumentation), xem [hướng dẫn Frida](./frida-instrumentation.md).

## Chọn hướng dẫn theo nhu cầu

| Tác vụ | Hướng dẫn |
| :--- | :--- |
| Nhận diện bản cài đặt phần mềm desktop | [Ứng viên runtime, bằng chứng và tệp theo dõi](desktop-applications.md) |
| Kiểm tra hàm native, tham chiếu hoặc kiểu dữ liệu | [Tệp nhị phân Native](native-investigation.md), [Metadata Go](go-binaries.md), [Phụ thuộc Rust](rust-binaries.md), [Tệp pyc CPython](python-pyc.md), [Chứng chỉ PE](pe-certificates.md), [Import PE](pe-imports.md), [Cấu hình tải/TLS PE](pe-load-config.md), [Ghi chú ELF](elf-notes.md), [Mô-đun Linux Kernel](kernel-modules.md), [Báo cáo sự cố Apple IPS](apple-ips.md), [Erlang BEAM](erlang-beam.md), [Bố cục ELF ngoại tuyến](binary-diagnostics.md), [Sự cố đã ghi](recorded-crashes.md) |
| Chọn provider native hoặc định dạng import | [Ngữ nghĩa Ghidra](ghidra-provider.md), [Windows](windows-ghidra-p0.md), [DOS](ghidra-dos.md), [Byte thô](raw-binary.md), [GameCube DOL](ghidra-dol.md), [MIPS](ghidra-mips.md), [PSP](ghidra-psp.md), [NativeAOT](ghidra-nativeaot.md), [IDA Pro](ida-provider.md), [Kuna](kuna-provider.md), [JEB](jeb-analysis.md) |
| Phân tích hiện vật JavaScript hoặc Electron | [Phân tích tĩnh](javascript-artifact-reconstruction.md), [Truy vết & so sánh](javascript-application-workflows.md), [Đồ thị ứng dụng](javascript-application-graph.md), [Khôi phục mã nguồn](javascript-recovery.md) |
| So sánh JavaScript tĩnh với quan sát runtime | [Gán nguồn runtime](javascript-runtime-observation.md), [Đối soát dữ liệu](javascript-runtime-reconciliation.md), [Quan sát Electron](electron-observation.md), [Cầu chì Electron fuses](electron-fuses.md) |
| Kiểm tra assembly, gói ứng dụng hoặc firmware | [Godot PCK](godot-pck.md), [UnityFS](unityfs.md), [U-Boot uImage](uimage.md), [.NET](managed-code-analysis.md), [Gói Python](python-packaged-applications.md), [Android APK](android-analysis.md), [Chữ ký APK](apk-signature.md), [Tài nguyên Apktool](apktool-resource-analysis.md), [Thu thập qua ADB](adb-device-analysis.md), [Apple iOS/macOS](apple-application-analysis.md), [HarmonyOS](harmony-analysis.md), [Firmware](firmware-analysis.md), [Ảnh ESP](esp-image.md), [Ảnh Flash ESP](esp-flash-image.md), [WASM](wasm-artifacts.md), [EVM Bytecode](evm-bytecode.md) |
| Quan sát hoặc so sánh kịch bản trình duyệt | [Quan sát qua CDP](browser-observation.md), [Kịch bản kịch bản](browser-scenario-contract.md), [Bằng chứng runtime web](web-runtime.md), [Bản ghi lưu trữ HAR/SAZ](web-network-captures.md) |
| Cung cấp hành vi đã ghi dưới dạng môi trường tác tử | [Môi trường tác tử (Agent Environments)](agent-environments.md) |
| Truy vết mã nguồn website đã thu thập | [Vị trí mã nguồn](web-source-location.md), [Dấu vết mô-đun](website-module-trace.md), [Xuất script](website-script-export.md) |
| Thu thập và so sánh tiến trình lệnh | [Bắt giữ tiến trình (Process Capture)](process-capture.md) |
| Sử dụng CLI/MCP và lưu giữ bằng chứng | [Hướng dẫn CLI](cli.md), [Hợp đồng MCP](mcp-contracts.md), [Lời nhắc tác tử (Agent Prompts)](mcp-prompts.md), [Sổ cái nghĩa vụ khôi phục](reconstruction-obligation-ledgers.md) |

Để phát triển mở rộng, xem [kiểm thử](testing.md), [xác minh client](client-verification.md),
[thiết kế công cụ](tool-design.md), [giới hạn tiếp nhận & khôi phục](design/limits-and-recovery.md),
và [phát hành](releasing.md).
[Độ sẵn sàng khôi phục](reconstruction-readiness.md) mô tả bộ kiểm tra sự phù hợp trong phát triển.
[Bản nâng cấp 6.2](migration-6.2.md) ghi lại các thay đổi tương thích; [Lộ trình phát triển](roadmap.md) theo dõi các tính năng tương lai.

![REA kiểm tra tệp nhị phân native trong Hopper](./assets/rea-hopper-analysis.png)
