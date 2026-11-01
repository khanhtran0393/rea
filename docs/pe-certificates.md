# Bảng chứng chỉ Authenticode PE (PE Authenticode Certificate Tables)

`inspect_pe_certificates` / `inspect-pe-certificates` trả lời câu hỏi liệu một tệp ảnh PE32/PE32+ cục bộ đã chọn có khai báo bảng `IMAGE_DIRECTORY_ENTRY_SECURITY` hay không, mục `WIN_CERTIFICATE` nào được lưu trữ trong bảng đó và những trường người ký PKCS#7 cùng X.509 nào mà các mục đó mang theo. Nó đọc tệp mà không cần thực thi và không yêu cầu Windows SDK, tệp catalog hay bộ tháo mã.

```bash
rea inspect-pe-certificates /duong/dan/tuyet/doi/toi/application.exe --json
```

CLI phân giải các đường dẫn tương đối dựa trên thư mục làm việc của bên gọi. MCP và lược đồ đầu vào được công bố yêu cầu đường dẫn hệ thống tệp tuyệt đối và bảo toàn lựa chọn chính xác đó:

```json
{
  "name": "inspect_pe_certificates",
  "arguments": { "path": "/artifacts/application.exe" }
}
```

## Quan sát (Observations)

Kết quả Bằng chứng (Evidence) nội tuyến chứa đường dẫn hiện vật, SHA-256 và độ dài byte. `table` là thư mục bảo mật theo độ lệch tệp, hoặc `null` khi thư mục bị thiếu hoặc bằng 0. Mỗi mục trong `entries[]` báo cáo độ dài `WIN_CERTIFICATE`, bản sửa đổi (revision), loại và SHA-256 của payload.

Các mục loại 2 có thể tính phần đệm căn chỉnh số 0 của chúng trong `dwLength`. Chủ sở hữu mục PE chỉ chấp nhận khoảng trống chính xác đến ranh giới 8 byte tiếp theo sau ContentInfo đã phân tích cú pháp; dữ liệu thừa hoặc phần đệm được đếm khác 0 sẽ thất bại. Phạm vi mục giữ lại tất cả các byte đã khai báo, bao gồm cả phần đệm được đếm. Phạm vi và mã băm payload CMS loại trừ phần đệm được đếm.

Đối với `WIN_CERTIFICATE` loại 2 (PKCS#7 signedData), `pkcs7` chứa các OID thuật toán băm (digest), các trường số sê-ri/nhà phát hành/chủ thể/hiệu lực X.509 được nhúng và các dữ kiện định danh người ký. Một người ký được xác định bởi định danh khóa chủ thể báo cáo giá trị đó dưới dạng `subject_key_identifier_hex`. Các loại chứng chỉ khác giữ `pkcs7` là null.

Mỗi người ký báo cáo `unsigned_attribute_oids`: null nghĩa là trường bị thiếu, và một mảng liệt kê các loại khi có mặt. Bằng chứng cũ được lưu giữ có thể bỏ qua trường này. Nội dung thuộc tính chưa ký, bao gồm chữ ký lồng nhau và mã token dấu thời gian, không được giải mã; sự hiện diện của chúng làm cho độ bao phủ trở thành một phần và thêm một giới hạn. Số lượng chứng chỉ và người ký đề cập đến SignedData bên ngoài.

Một bảng bị thiếu là sự vắng mặt quan sát được của thư mục đó. Đó không phải là bằng chứng tệp ảnh không được ký trong mọi danh mục hoặc overlay.

## Giới hạn và lỗi (Limits & Failures)

Tệp được chọn được giới hạn ở 64 MiB theo mặc định (tối đa 512 MiB). Tối đa tám mục `WIN_CERTIFICATE` được kiểm tra theo mặc định. Header PE bị lỗi, mục chứng chỉ bị cắt cụt và PKCS#7 không phải là signedData về mặt cấu trúc sẽ thất bại mà không có thành công một phần. Thư mục bảo mật kết thúc với ít hơn 8 byte header, bao gồm các số 0, là `WIN_CERTIFICATE` bị cắt cụt và thất bại theo cùng một cách. SignedData phải chứa ít nhất một `SignerInfo`, và mỗi `SignerInfo` phải mang một giá trị chữ ký không rỗng. Mỗi chứng chỉ X.509 được nhúng phải bao gồm `subjectPublicKeyInfo` và `signatureValue`. Văn bản tên chứng chỉ đã giải mã, số sê-ri, cách viết hiệu lực và định danh khóa chủ thể chia sẻ ngân sách 64 KiB cho mỗi payload PKCS#7, và mỗi payload giải mã tối đa 16.384 giá trị ASN.1. Việc cạn kiệt một trong hai mức cho phép cố định sẽ trả về lỗi `resource_constraint` không thể thử lại nêu tên nó, không phải lỗi đầu vào không hợp lệ; bản phát hành này không có cơ chế ghi đè runtime, và `max_file_bytes` cùng `max_entries` không nâng mức này lên. Các mã hóa tên chứng chỉ không được hỗ trợ, bao gồm GeneralString, sẽ thất bại rõ ràng. Các giá trị TeletexString (T.61) được hiển thị với ánh xạ tương thích ISO-8859-1 được sử dụng bởi OpenSSL; đây không phải là bộ giải mã T.61 đầy đủ, vì vậy kết quả bao gồm một giới hạn bất cứ khi nào ánh xạ đó được sử dụng. RFC 5280 giữ lại TeletexString để tương thích ngược và mô tả danh mục ký tự của nó tương tự như Latin giống ASCII với các ký tự bổ sung.

Bộ đọc CMS cấu trúc chấp nhận các độ dài xác định và độ dài bất định BER cho các giá trị được xây dựng, bao gồm cả chuỗi `ContentInfo` bên ngoài. Các giá trị nguyên thủy phải có độ dài xác định. Các chứng chỉ X.509 được nhúng và các thuộc tính đã ký cũng phải sử dụng mã hóa độ dài xác định, theo yêu cầu của hồ sơ DER của chúng. Đây là một tập hợp con cấu trúc có giới hạn: nó không triển khai mọi dạng chuỗi BER hoặc xác thực mã hóa chuẩn DER, và nó không xác minh chữ ký.

Quy trình làm việc này không xác minh chữ ký, chuỗi chứng chỉ, dấu thời gian hoặc độ bao phủ băm Authenticode của các byte tệp ảnh.
