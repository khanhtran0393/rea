# Gói tệp đơn lẻ .NET (Single-File Bundles)

Sử dụng `rea inspect-artifact /absolute/path/host --json` để kiểm kê máy chủ tệp đơn lẻ .NET thông thường và `rea extract-artifact /absolute/path/host --json` để hiện thực hóa các thành viên của nó vào một thư mục đầu ra mới do REA sở hữu. Với máy chủ được chọn qua `open_binary`, các công cụ MCP tương ứng là `inspect_artifact` và `extract_artifact`. Không cần cài đặt .NET để kiểm tra hoặc trích xuất.

Đối với kiểm kê thu gọn, hãy gọi `inspect_artifact` với `detail: "summary"`, sau đó sử dụng `inspect_analysis_view` với `parent_evidence_id` được trả về. Các trang xuất hiện kiểm kê chấp nhận `logical_paths`, `occurrence_ids`, `fields` đã chọn, `offset`, và `limit`; các trang node chấp nhận các lô `artifact_ids` để truy xuất mã băm byte mở rộng. Bao gồm `format_metadata` và `source_location` khi các khai báo thành viên và phạm vi lưu trữ quan trọng. Các trường danh tính tồn tại qua phép chiếu và Bằng chứng kiểm tra hoàn chỉnh vẫn được giữ lại.

REA phát hiện dấu hiệu apphost và độ lệch tiêu đề đứng trước của nó từ các byte PE, ELF và Mach-O mỏng, bất kể tên tệp. Độ lệch bằng 0 của apphost chưa đóng gói vẫn là một artifact native thông thường. Kiểm kê lát cắt Mach-O phổ quát giữ nguyên hành vi hiện có của nó; nội dung đi kèm bên trong các lát cắt không được mở rộng. Khi một tệp native có CArchive PyInstaller được nhận diện, việc quét dấu hiệu apphost sẽ dừng lại trước tải trọng của nó. Một trình trợ giúp .NET được lưu trữ bên trong kho lưu trữ đó không thay thế định dạng của container bên ngoài; một máy chủ .NET bên ngoài vẫn có thể chứa một trình trợ giúp PyInstaller dưới dạng thành viên.

## Bố cục được hỗ trợ

| Tiêu đề | Thế hệ trình tạo                   | Mã hóa thành viên           |
| ------- | ----------------------------------- | --------------------------- |
| 1.0     | .NET Core 3.x                       | Không nén                   |
| 2.0     | .NET 5                              | Không nén                   |
| 6.0     | .NET 6 và mới hơn sử dụng bố cục này| Không nén hoặc raw DEFLATE  |

Các phiên bản, phiên bản phụ và cờ nằm ngoài tập hợp này đều bị từ chối rõ ràng; chúng không âm thầm trở thành một PE thông thường hoặc một bản kiểm kê rỗng thành công. Cờ 0 và cờ tương thích v2+ 1 được tiếp nhận. Byte loại tệp không xác định được giữ lại với một giới hạn, thay vì được coi là thẩm quyền thực thi. Các wrapper được mã hóa/bảo vệ và việc khôi phục tải trọng của chúng nằm ngoài phạm vi của trình đọc này.

Máy chủ giữ SHA-256 toàn bộ tệp của nó. Mỗi tệp nhúng nhận SHA-256 byte mở rộng, đường dẫn, kích thước khai báo, kích thước nén và khai báo `format_metadata` (`major_version`, `minor_version`, `file_type`). `source_location` của nó mô tả phạm vi máy chủ được lưu trữ, bao gồm độ dài nén khi áp dụng. `root_format` của manifest gốc là `dotnet-bundle`; node nội dung gốc vẫn ghi lại định dạng PE/ELF/Mach-O native của nó. Khai báo loại tệp hoặc phần mở rộng `.dll` không xác lập một assembly hợp lệ. Hãy kiểm tra rõ ràng một thành viên được trích xuất bằng các công cụ managed để xác lập các dữ kiện PE/CLI.

## Tiếp nhận và ngăn chặn

Metadata được đọc theo dải với bộ đệm 64 KiB. Ngân sách chính sách là 16 MiB metadata manifest, 100.000 tệp, 64 KiB cho mỗi chuỗi UTF-8, 512 MiB byte được lưu trữ hoặc mở rộng cho mỗi tệp, 256 phần đường dẫn cho mỗi thành viên, 250.000 node cây đường dẫn riêng biệt trong bộ nhớ, và 2 GiB tổng số byte mở rộng. Ngân sách node chỉ đếm tiền tố thư mục dùng chung một lần. Đây là các chính sách tài nguyên của REA, không phải giới hạn do định dạng của Microsoft áp đặt. Quét dấu hiệu truyền các đoạn 64 KiB và tái sử dụng các dữ kiện thăm dò đã hoàn thành trong quá trình trích xuất; các tệp native thông thường không có dấu hiệu sẽ phát sinh thêm một lượt đọc trước khi băm máy chủ.

Quá trình trích xuất băm lại toàn bộ máy chủ thông qua cùng bộ mô tả được sử dụng để đọc các thành viên và so sánh nó với mã băm gốc của bản kiểm kê trước khi tạo thư mục đầu ra. Điều này thêm một lượt đọc toàn bộ máy chủ vào quá trình trích xuất, bao gồm cả khi bản thân các thành viên nhúng nhỏ.

Tiêu đề bị lỗi/bị cắt cụt, đường dẫn không an toàn hoặc xung đột, dải thành viên chồng chéo, các thành viên cắt ngang manifest, phạm vi cấu hình mâu thuẫn, UTF-8 không hợp lệ, dữ liệu nén bị hỏng/dư thừa và kích thước mở rộng không chính xác sẽ thất bại trước khi xuất bản đầu ra. Danh tính bộ mô tả/đường dẫn nguồn được kiểm tra trong khi đọc; trích xuất xác minh mã băm máy chủ và các byte thành viên đối chiếu với bản kiểm kê. Các gốc đầu ra hiện có không bao giờ bị ghi đè. Việc hủy bỏ và lỗi sẽ giải phóng các luồng/bộ mô tả thuộc sở hữu thông qua phạm vi khôi phục trích xuất dùng chung.

## Tái tạo fixture của trình tạo

Trình tạo thuộc mã nguồn sử dụng trình tạo **Microsoft HostModel** của .NET 8 SDK đã cài đặt để phát ra các gói x64 v1, v2, v6 thuần và v6 nén cho Windows, Linux hoặc macOS. Nó không chạy các máy chủ được tạo ra hoặc tải xuống các gói runtime. Cung cấp mẫu apphost SDK từ `packs/Microsoft.NETCore.App.Host.<rid>/<version>/runtimes/<rid>/native/apphost` (`apphost.exe` trên Windows). Truyền `windows` (mặc định), `linux`, hoặc `osx` làm đối số trình tạo thứ ba tùy chọn để khớp với mẫu đó.

```bash
dotnet build scripts/fixtures/dotnet-bundle/BundleFixture.csproj -c Release --configfile scripts/fixtures/dotnet-bundle/NuGet.Config -o /absolute/path/producer -p:BaseIntermediateOutputPath=/absolute/path/producer-obj/
dotnet /absolute/path/producer/BundleFixture.dll /absolute/path/apphost.exe /absolute/path/absent-bundles-directory
rea inspect-artifact /absolute/path/absent-bundles-directory/net8-compressed/Probe.exe --json
rea extract-artifact /absolute/path/absent-bundles-directory/net8-compressed/Probe.exe --json
```

Trình tạo ghi `oracle.json` bên cạnh mỗi máy chủ. Một lượt `BinaryReader`/`DeflateStream` C# độc lập ghi lại phiên bản, loại thành viên và phạm vi được lưu trữ, sau đó so sánh mã băm của từng thành viên mở rộng với tệp nguồn của nó. So sánh các bản ghi đó với Bằng chứng kiểm tra và các byte được trích xuất. Điều này xác minh trình ghi nén của Microsoft tách biệt khỏi các fixture phân tích cú pháp tổng hợp; nó không phải là bằng chứng cho thấy máy chủ đi kèm sẽ thực thi trên một runtime mục tiêu.
