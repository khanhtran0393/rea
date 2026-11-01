# Provider Kuna

[Kuna](https://github.com/Noelo-Lab/kuna) là một trình dịch ngược theo giấy phép Apache-2.0 được phân phối dưới dạng một tệp thực thi `kuna`. Đối với bản cài đặt mới, hãy sử dụng Kuna 1.986, bản phát hành đã được xác minh được REA khuyến nghị. REA sử dụng tệp `kuna` do bạn cung cấp; nó không tải xuống, cài đặt, cập nhật hoặc đóng gói sẵn Kuna.

Kuna là tĩnh: nó đọc hình ảnh đã chọn và không bao giờ thực thi nó.

## Cấu hình

```bash
export REA_KUNA_COMMAND=/absolute/path/to/kuna
rea doctor --provider kuna
```

Chọn nó một cách rõ ràng với `--provider kuna`, `open_binary.provider_id: "kuna"` của MCP, hoặc `REA_ANALYSIS_PROVIDER=kuna`.

| Thiết lập                     | Mặc định                                                                                               | Ý nghĩa                                                                 |
| ----------------------------- | ------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------- |
| `REA_KUNA_COMMAND`            | không có (bắt buộc)                                                                                    | Đường dẫn tuyệt đối của tệp thực thi `kuna`.                            |
| `REA_KUNA_CACHE_DIR`          | `~/Library/Caches/rea/kuna` (macOS), `$XDG_CACHE_HOME/rea/kuna` (Linux)                                | Thư mục tuyệt đối REA sở hữu cho các bản xuất dự án.                    |
| `REA_KUNA_CACHE_MAX_BYTES`    | 4 GiB                                                                                                  | Giới hạn kích thước cache.                                              |
| `REA_KUNA_PROJECT_TIMEOUT_MS` | 30 phút                                                                                                | Hạn chót cho việc kiểm kê/xuất toàn bộ chương trình, quét chuỗi và xref.|
| `REA_KUNA_TIMEOUT_MS`         | 120 giây                                                                                               | Hạn chót cho việc dịch ngược một hàm đơn lẻ.                            |
| `REA_KUNA_JOBS`               | 1                                                                                                      | Số lượng luồng công việc `decompile-project --jobs`.                    |

Kuna chưa hỗ trợ Windows. Các mục tiêu được hỗ trợ là tệp thực thi ELF và Mach-O mỏng cho x86-64, AArch64 và ARM; PE và Mach-O phổ quát bị từ chối.
