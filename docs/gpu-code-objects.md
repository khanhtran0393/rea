# Đối tượng mã CUDA và HIP tĩnh (GPU Code Objects)

`inspect_gpu_code_objects` trả lời các mục tiêu GPU và khai báo kernel nào mà một artifact cục bộ mang theo. Lệnh đọc các section ELF/PE, fatbin CUDA độc lập, CUBIN/PTX và bundle Clang HIP, bao gồm cả CCOB v1/v2/v3. Nó không thực thi artifact hay gọi các công cụ CUDA/ROCm. Việc thực thi GPU, dịch ngược SASS và tương quan khởi chạy máy chủ nằm ngoài phạm vi bao phủ.

Bắt đầu với một bản kiểm kê mục tiêu:

```json
{ "path": "/artifacts/library.so", "view": { "collection": "targets" } }
```

Các hàng mục tiêu tổng hợp các tải trọng đã kiểm kê và phân biệt phân tích kernel chưa được yêu cầu với phân tích hoàn chỉnh hoặc một phần. Các trang `objects` hiển thị ID đối tượng chính xác, khai báo nén và vị trí đã mã hóa/tiêu đề.

`target` là khai báo kiểm kê được sử dụng để lựa chọn; `payload_target` là khai báo bên trong được quan sát độc lập khi có sẵn. Các kiến trúc cơ sở mâu thuẫn giữ lại cả hai giá trị, đánh dấu độ bao phủ đối tượng/báo cáo/mục tiêu là một phần và chú thích các kernel bị ảnh hưởng.

ID mục tiêu HIP xác định mục tiêu chuỗi công cụ, không phải biểu diễn tải trọng của nó. Bitcode LLVM thô hoặc được bọc được nhận diện vẫn ở trạng thái `unsupported` đối với việc kiểm tra metadata kernel.

CCOB được nhận diện bởi 4 byte chữ ký chính xác `CCOB`. CCOB v1 không lưu trữ tổng kích thước: luồng zlib hoặc Zstd đã hoàn thành xác định phạm vi mã hóa của nó.

Giải mã các mục tiêu đã chọn và trả về một trang kernel trong cùng một lệnh gọi:

```json
{
  "path": "/artifacts/library.so",
  "analysis": "kernels",
  "targets": ["sm_100"],
  "view": {
    "collection": "kernels",
    "names": ["kernel_name"],
    "fields": [
      "name",
      "parameter_count",
      "parameter_bytes",
      "parameters",
      "registers",
      "shared_bytes"
    ]
  }
}
```
