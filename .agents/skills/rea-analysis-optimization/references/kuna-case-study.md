# Kuna: Tái sử dụng trước khi thêm máy móc phức tạp (Kuna: reuse before adding machinery)

Đọc tài liệu tham khảo này để biết các ví dụ cụ thể về các quyết định của kỹ năng. Các bản ghi dưới đây liên quan đến phân tích tĩnh trên Linux với Kuna do bên gọi tự cung cấp. Chúng không xác lập tính tương đương với một provider khác hoặc lợi thế chung cho Code Mode, các công cụ nguyên tử, hay bất kỳ thiết kế điều phối cụ thể nào.

## Tra cứu địa chỉ chính xác: một cải tiến hẹp, có đo lường (Exact-address lookup: a measured, narrow improvement)

[PR #4695](https://github.com/morluto/rea/pull/4695) đã tái sử dụng thứ tự địa chỉ số của kho kiểm kê chuỗi đã được xác thực. Trước đây, mỗi lệnh `list_strings(address)` đều lọc qua tất cả các dữ kiện được lưu trong bộ nhớ đệm. Chủ thể sở hữu hiện thực hiện tra cứu cận dưới nhị phân (binary lower-bound lookup) và trả về mọi mã hóa tại địa chỉ đó: O(log n + số kết quả khớp), mà không cần chỉ mục tra cứu lưu giữ thứ hai.

Triển khai sở hữu là [`KunaStringInventory.ts`](../../../../src/kuna/KunaStringInventory.ts), được tiêu thụ bởi [`KunaStringQueries.ts`](../../../../src/kuna/KunaStringQueries.ts). Kiểm tra các phiên bản hiện tại của chúng trước khi áp dụng ví dụ. Các địa chỉ số u64, mã hóa cùng địa chỉ, từ chối trùng lặp, mã băm nguồn, lựa chọn của bên gọi và vòng đời phiên hiện có là một phần của hợp đồng. Toàn bộ danh sách và các tìm kiếm literal/regex vẫn giữ nguyên ngữ nghĩa của chúng. Một nỗ lực loại bỏ tập hợp phát hiện trùng lặp đã bị hủy bỏ: không có lợi ích bộ nhớ ổn định nào được xác lập và nó làm chậm việc từ chối đầu vào bị sai định dạng.

### Khối lượng công việc và kết quả được ghi nhận (Recorded workload and results)

Benchmark đã sử dụng Kuna 1.833 chưa sửa đổi và LLVM 16 trên Linux x86-64, với 602.544 dữ kiện chuỗi. Mốc cơ sở là `be0b2d138`; tính năng cuối cùng là `5f564962da0e1b17f00a68a74a312539f0209a01`, sau đó được tích hợp trong `c6b62754b095a134d6d67877508fa34311a2eda2`. Đây là các quan sát lịch sử, không phải là benchmark cho mọi phiên bản main tiếp theo.

Cùng một tác vụ MCP stdio production đã khám phá danh mục, mở tệp nhị phân, thực hiện một lần đọc chọn lọc khi cold và 28 lần đọc chính xác khi warm, kiểm tra Evidence được lưu giữ và đóng phiên. Năm địa chỉ được chọn và hai bộ chọn vắng mặt được đọc bốn lần. Quá trình quét lọc native đã chọn các chuỗi; một ánh xạ tiêu đề tải ELF độc lập đã kiểm tra các byte nguồn và ký tự kết thúc. Các dữ kiện được trả về, liên kết nguồn, mã băm mục tiêu và quá trình dọn dẹp đã được kiểm tra.

Ba phiên mới luân phiên cho mỗi triển khai đã sử dụng 1 CPU, giới hạn cứng nhóm tiến trình 4 GiB, dừng sớm ở 3 GiB và không dùng swap. Bộ nhớ đệm tệp hệ điều hành là warm. Toàn bộ chi phí bao gồm oracle native độc lập, băm, công việc danh mục/client và kiểm tra tính đúng đắn.

| Trung vị (Median) | Trước | Sau |
| :--- | :---: | :---: |
| 28 lần đọc MCP chính xác khi warm | 495.2 ms | 90.8 ms |
| Lần đọc MCP chọn lọc khi cold | 11.30 s | 11.01 s |
| Toàn bộ thời gian trôi qua | 28.24 s | 27.43 s |
| Toàn bộ CPU nhóm tiến trình | 27.71 s | 26.89 s |
| Đỉnh nhân hệ điều hành (kể cả file cache) | 2237.6 MiB | 2240.6 MiB |

Tổng thời gian warm dao động từ 491.7–544.3 ms trước đó xuống 84.8–99.5 ms sau đó: nhanh hơn khoảng 5.5 lần ở giai đoạn này. Thời gian cold/toàn tác vụ và bộ nhớ chồng chéo nhau; không có cải thiện ổn định nào trong các chi phí đó được xác lập. Một cặp SQLite nhỏ (7.985 dữ kiện) đo được 20 lần đọc warm ở mức 88.6 → 80.5 ms và toàn bộ quá trình ở mức 13.68 → 13.94 s. Lợi ích tra cứu kho kiểm kê lớn không khái quát hóa thành việc tăng tốc toàn bộ tác vụ trên các đầu vào nhỏ.

Mã băm danh mục, số lượng lệnh gọi và các dữ kiện chuẩn hóa không thay đổi. Ước tính `cl100k_base` qua một lượt JSON danh mục thô, đối số và câu trả lời là khoảng 846.000 token trong cả hai luồng, với khoảng 802.562 từ danh mục. Đây là tuần tự hóa truyền tải, không phải ngữ cảnh mô hình được hiển thị hoặc mức sử dụng thực tế; việc khám phá của client, prompt, phát lại và đóng khung giao thức chưa được đo lường. **Không có sự giảm thiểu token hoặc kết quả điều tra tác tử được cải thiện nào được xác lập.**

SHA-256 mục tiêu được ghim:
`f62d254b7f2bf42df8c8b07d46ee3bb4c2cafeca436b2e6bc6ccbe4581f58f40`.
SHA-256 engine được ghim:
`d858918b0766435cd96b999a77f2d5a31a0cc4f07256421c883b5b9dc4dd58e4`.
Cả sáu mã băm chuỗi kết quả chuẩn hóa đều là:
`4d2f90269da8568984090b20274163c4a7c2a0538aab63530bd2fe44840a0bba`.
Sự trùng khớp byte đã chọn không chứng minh tính đầy đủ của khám phá native; các chuỗi và mối quan hệ bị thiếu có thể vẫn là chưa xác định.

## Các thay đổi có thể chuyển giao khác và giới hạn của chúng (Other transferable changes and their limits)

- [Kho kiểm kê chuỗi và tìm kiếm được lưu trong bộ nhớ đệm, #4095](https://github.com/morluto/rea/pull/4095): chia sẻ các dữ kiện đã xác thực qua các truy vấn trong một phiên xác định. Kiểm toán các đầu vào chính và vòng đời trước khi mở rộng việc tái sử dụng trên các mục tiêu hoặc phiên bản engine.
- [Công việc hàm đơn dùng chung, #4045](https://github.com/morluto/rea/pull/4045) và [tái sử dụng yêu cầu lặp lại, #4169](https://github.com/morluto/rea/pull/4169): tránh công việc native trùng lặp trong khi vẫn giữ quyền sở hữu dọn dẹp và hủy cho mỗi bên gọi. Chỉ riêng số lượng truy vấn không thể tiết lộ các tiến trình con trùng lặp.
- [Chuỗi được truyền luồng (streamed strings), #4318](https://github.com/morluto/rea/pull/4318) và [đồ thị lệnh gọi truyền luồng, #4329](https://github.com/morluto/rea/pull/4329): loại bỏ các lỗi bộ đệm truyền tải tổng hợp cố định thông qua việc phân tích cú pháp tăng dần. Việc truyền tải theo luồng không giới hạn mọi đối tượng được lưu giữ hoặc câu trả lời cuối cùng. Các liên kết này xác lập các thay đổi; không có khẳng định tăng tốc số lượng nào ở đây.

Các dữ kiện tham chiếu có kiểu, phép chiếu truy vấn chuỗi native và các lần đọc byte native dùng chung vẫn là thử nghiệm tại mốc 2026-10-17 `65f2f9a22bc892f61d2b00f9a4611124f57d4bff` trên `experiment/atomic-codemode-20261016`. Một số thử nghiệm đã sử dụng engine native đã được vá. Hãy kiểm tra tích hợp hiện tại và khả năng của engine tự cung cấp trước khi đề xuất chúng; một thử nghiệm hoạt động không phải là một bảo đảm của provider tiêu chuẩn.

## Áp dụng lập luận vào nơi khác (Applying the reasoning elsewhere)

Đối với một provider hoặc định dạng khác, hãy kiểm tra biểu diễn thực tế và khả năng native của nó trước khi mượn triển khai. Thứ tự số hiện có có thể hỗ trợ các lần đọc phạm vi; một truy vấn native trên đĩa có thể tránh việc hiện thực hóa kho kiểm kê; một lần phân tích bất biến có thể hỗ trợ một số chế độ xem. Đo lường phương án thay thế liên quan thay vì yêu cầu mọi provider phải sử dụng mảng được lưu trong bộ nhớ đệm của Kuna.

Đối với các quan sát trực tiếp, các quy tắc lưu bộ nhớ đệm hiện vật bất biến là không đủ: xác thực danh tính tiến trình/phiên và độ tươi mới theo yêu cầu của bên gọi. Đối với các câu trả lời rút gọn, hãy duy trì quyền truy cập vào Evidence nguồn và báo cáo độ bao phủ đã kiểm tra; việc chiếu không được âm thầm biến một cạnh chưa được giải quyết thành không có cạnh. Đối với việc xử lý hàng loạt, hãy kiểm tra các thành công và thất bại hỗn hợp và đo lường toàn bộ tác vụ, bao gồm khám phá, các lệnh gọi bổ sung, công việc native và tuần tự hóa.
