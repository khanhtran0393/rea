# Phân tích Firmware

REA chấp nhận **Binwalk 3.1.x** để kiểm tra vùng và **Unblob 26.6.x** để trích xuất rõ ràng. Trình phân tích cú pháp báo cáo được xác minh với Binwalk 3.1.0 và Unblob 26.6.4. Cột mốc firmware đầu tiên này được xác minh trên Linux x64. Không cần thiết bị, quyền root, mount, giả lập hay thực thi các chương trình đã trích xuất.

## Các công cụ do bên gọi cung cấp

Cung cấp các công cụ hiện có bằng đường dẫn thực thi tuyệt đối:

```sh
export REA_BINWALK_COMMAND=/absolute/path/to/binwalk
export REA_UNBLOB_COMMAND=/absolute/path/to/unblob
```

REA không cài đặt các công cụ này hay phụ thuộc của chúng. Nó kiểm tra biểu ngữ `--version` của công cụ khi thực thi yêu cầu. Cả hai đều yêu cầu `prlimit` của util-linux (`/usr/bin/prlimit` hoặc qua `REA_FIRMWARE_PRLIMIT_COMMAND`).

Unblob cần bộ trích xuất bên ngoài cho định dạng đã chọn trên `PATH` được kế thừa: ví dụ: `debugfs` cho ext2/ext3/ext4. Hộp cát Landlock mặc định của nó vẫn được bật.

## CLI và MCP

```sh
rea inspect-firmware-regions /path/to/firmware.bin
rea extract-firmware /path/to/firmware.bin /path/to/new-output
rea extract-firmware /path/to/firmware.bin /path/to/selected-output   --offset 64 --length 4096 --max-depth 3
```

Các công cụ MCP tương ứng chấp nhận cùng mục đích.
