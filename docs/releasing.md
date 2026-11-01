# Phát hành REA (Releasing REA)

Các lần đẩy (push) lên nhánh main sẽ tự động mở hoặc làm mới một PR Release Please với phiên bản và changelog tiếp theo. Xem xét các ghi chú di chuyển của nó và đợi các kiểm tra CI hiện tại của nó đạt chuẩn. Việc hợp nhất PR đó vào main sẽ tự động bắt đầu quá trình xuất bản.

Quy trình phát hành xác thực metadata của bản phát hành đã hợp nhất trước khi tạo tag. Việc xuất bản lên npm và MCP Registry sử dụng chính xác SHA hợp nhất đã được kiểm duyệt do Release Please trả về. Trước khi xuất bản lên một trong hai registry, lượt chạy phát hành sẽ gọi đánh giá đủ điều kiện CI đầy đủ tại chính xác bản sửa đổi đó, bao gồm độ bao phủ, các provider khả chuyển và mọi máy chủ lưu trữ gói native.

Sau khi đủ điều kiện, một runner riêng biệt với quyền chỉ đọc đối với kho lưu trữ/actions sẽ khôi phục tarball và runtime của bộ xác minh được giữ lại. Nó cài đặt chính xác kho lưu trữ đó vào một tiền tố trống, sau đó thực hiện kiểm tra trợ giúp/khám phá CLI và hợp đồng phiên/danh mục stdio MCP thực tế.

Chỉ khi buổi diễn tập thành công thì job có thẩm quyền xuất bản npm mới được kích hoạt. Trình xuất bản gọi `npm publish --ignore-scripts` đối chiếu với registry chính thức.

## Khôi phục một bản phát hành npm đã xuất bản

Khi một bộ xác minh hoặc quy trình xuất bản cần sửa mã, hãy đưa bản sửa lỗi đó vào main trước, sau đó sử dụng quy trình khôi phục riêng biệt thay vì lặp lại việc tạo bản phát hành:

```bash
gh workflow run release-recovery.yml --ref main   -f version=6.5.0 -f release_run_id=38564502044
```

Chọn phiên bản hiện có và lượt chạy Release tự động ban đầu của nó. Phục hồi yêu cầu cổng CI nguồn chính xác thành công của lượt chạy đó cộng với ghi npm hoặc xác minh byte registry thành công. Chỉ sau khi xác minh thành công, phục hồi mới xuất bản `server.json` đã phát hành lên MCP Registry.
