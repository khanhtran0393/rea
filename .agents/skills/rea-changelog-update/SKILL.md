---
name: rea-changelog-update
description: Chuẩn bị hoặc viết lại changelog phát hành của REA và ghi chú phát hành GitHub từ lịch sử Git đã ghim, với lời cảm ơn người đóng góp đã được xác minh và đồng bộ hóa Release Please.
---

# Cập nhật changelog REA (REA changelog update)

Sử dụng kỹ năng này để chuẩn bị ghi chú phát hành (release notes) và ghi nhận đóng góp (contributor attribution). Đọc [docs/releasing.md](../../../docs/releasing.md) từ thư mục gốc để biết quyền hạn phát hành và các kiểm tra điểm mốc hiện có. Kỹ năng này chỉ chuẩn bị ghi chú; nó không cấp quyền merge, tạo tag hoặc phát hành gói.

## Kiểm kê trước khi viết (Inventory before prose)

Tìm nạp lịch sử và các tag đầy đủ. Chọn tag mốc đã phát hành từ `.release-please-manifest.json` trên **nhánh gốc (base branch)** của PR phát hành, và ghim SHA chính xác của nhánh đó làm mục tiêu biên tập. Các tag REA có định dạng `rea-agents-VERSION`. Sử dụng phiên bản đề xuất, bao gồm cả hậu tố prerelease; không thay đổi chiến lược `always-bump-minor` để bắt chước phiên bản lịch của OpenClaw.

```bash
node scripts/release-notes.mjs inventory \
  --repository morluto/rea --base rea-agents-PREVIOUS \
  --target FULL_BASE_BRANCH_SHA --version VERSION \
  --manifest /tmp/rea-VERSION.contributions.json
node scripts/release-notes.mjs record \
  --manifest /tmp/rea-VERSION.contributions.json \
  --summary \
  --output /tmp/rea-VERSION.record.md
```

Đọc bản kiểm kê, không chỉ đọc bản nháp của bot. Nó bao gồm các commit nhánh phụ, PR liên kết trên GitHub có commit merge trong phạm vi, tác giả/đồng tác giả commit và các issue đã đóng được xác nhận cùng người báo cáo. Kết nối GitHub được phân trang. Lỗi mạng sẽ dừng tiến trình; hãy chạy lại vào tệp đầu ra mới sau khi phục hồi. Các tệp đầu ra hiện có tuyệt đối không bị ghi đè.

Các định danh commit chưa giải quyết được giữ lại dưới dạng tên/email với `user: null`; không bao giờ đoán handle hoặc cảm ơn người merge để thay thế. Báo cáo bảo toàn các điểm chưa xác định. Đây là bản chụp metadata đã được xem xét, không phải bằng chứng độc lập rằng con người đã tạo ra bản vá. Đối với các danh tính không có thông tin, chiến lược merge bất thường, backport, hoàn tác hoặc tham chiếu mà GitHub không liên kết, hãy kiểm tra PR và diff gốc rồi báo cáo giới hạn. Trình thu thập hiện tại sử dụng các mối quan hệ đóng PR đã được xác nhận để ghi nhận công lao của người báo cáo issue; các đề cập issue không liên quan không được tự động ghi nhận. Không khẳng định rằng công việc chỉ mới được kiểm kê vẫn được xuất xưởng sau khi bị revert.

## Viết cho người dùng và giữ nguyên ghi nhận (Write for users and retain credit)

Chỉ viết lại phần `## [VERSION]` mới nhất trong `CHANGELOG.md`, bảo toàn tiêu đề phiên bản/ngày/liên kết của nó. Sử dụng các mục danh sách Markdown một dòng dưới:

- `### Highlights`: một vài kết quả định hình bản phát hành cho người dùng; không có hạn ngạch cho các bản phát hành nhỏ.
- `### Changes`: các khả năng mới và thay đổi hành vi có ý nghĩa.
- `### Fixes`: các bản sửa lỗi về độ tin cậy, tính đúng đắn, khả năng tương thích và an toàn.
- `### ⚠ BREAKING CHANGES`: giữ lại khi áp dụng, kèm theo hướng dẫn di chuyển (migrations) và tham chiếu PR/commit gốc được yêu cầu bởi trình xác thực điểm mốc hiện có.

Gom nhóm công việc liên quan theo quy trình làm việc của nhà phân tích hoặc tác động đến người dùng. Giải thích hành vi kết quả trong các mục ngắn gọn, cụ thể. Highlights tóm tắt bản phát hành; tránh lặp lại mô tả đầy đủ của chúng trong Changes. Dọn dẹp nội bộ, kiểm thử, CI và tài liệu định kỳ thường chỉ thuộc về tệp nguồn gốc (provenance file). Đọc các PR/diff mơ hồ trước khi đưa ra nhận định. Phân biệt giữa các kiểm tra provider mô phỏng và xác minh provider thực tế.

Ghi nhận mỗi đóng góp đại diện một lần duy nhất trong ghi chú công khai kèm liên kết của nó và `Thanks @...` cho tác giả con người đã được xác minh, các đồng tác giả đã biết và người báo cáo issue đã đóng được xác nhận. Việc gom nhóm giữ lại toàn bộ các ghi nhận đó trên một dòng. Khi Highlights, các mục chi tiết hoặc hướng dẫn di chuyển tham chiếu đến cùng một công việc, hãy đặt lời cảm ơn vào một trong các mục đó, không phải tất cả. Bao gồm các liên kết issue. Các gạch đầu dòng commit trực tiếp mang liên kết commit và ghi nhận con người đã biết. Loại trừ bot GitHub và các tài khoản tác tử đã biết. Các trailer đồng tác giả Anthropic, Cursor, Codex và Copilot quan sát được của REA giải quyết thành `@claude`, `@cursoragent`, `@codex` và `@Copilot` với loại GitHub `User`; chính sách ghi nhận loại trừ chúng trong khi vẫn giữ nguyên metadata của chúng. Theo yêu cầu của người duy trì REA, bỏ qua việc tự cảm ơn `@morluto` trong khi vẫn bảo toàn quyền tác giả trong bản kiểm kê. Các cộng tác viên con người khác vẫn đủ điều kiện. Sử dụng liên kết GitHub rõ ràng cho các tham chiếu liên kho lưu trữ để tránh các số `#NNN` mơ hồ. Các handle chưa biết vẫn giữ nguyên là chưa biết; chúng không phải là sự ghi nhận bịa đặt.

Nối liên kết nguồn gốc `record --summary` được tạo nguyên văn vào cuối bản phát hành này. Bản kiểm kê được liên kết giải trình cho mọi PR và đóng góp trực tiếp trong phạm vi được phát hiện, bao gồm cả công việc nội bộ bị bỏ sót trong văn bản. Không dán toàn bộ đầu ra `record` vào ghi chú công khai: nó làm trùng lặp lời cảm ơn và che khuất các thay đổi hướng tới người dùng. Các bản ghi đầy đủ vẫn có sẵn để xem xét. Không chỉnh sửa thủ công các ghi nhận được tạo tự động để vượt qua xác thực. Làm mới bản kiểm kê khi lịch sử sản phẩm thay đổi; việc sửa chữa văn bản thuần túy có thể tái sử dụng bản chụp.

## Hoàn thiện với Release Please (Finalize with Release Please)

Trước khi thay đổi PR phát hành trực tiếp, hãy đảm bảo có ủy quyền cho bản cập nhật PR đó. Việc chuẩn bị và xác thực cục bộ có thể tiến hành độc lập.

1. Thêm nhãn `rea:release-notes-finalized` vào PR phát hành đang mở được chọn **trước khi** chỉnh sửa nhánh/nội dung của nó. Các lượt push đề xuất chính tôn trọng nhãn này. Đây là lớp bảo vệ quy trình làm việc của chúng tôi, không phải cơ chế `autorelease: snooze` của Release Please. Một đề xuất đang chạy phải kết thúc trước khi chỉnh sửa. Không gửi bản chuẩn bị thủ công đè lên các ghi chú đã hoàn thiện.
2. Trên nhánh phát hành đó, lưu bản kiểm kê đã xem xét dưới dạng `docs/releases/VERSION.contributions.json`. Đây là nguồn gốc phát hành công khai lâu dài, không phải tệp lập kế hoạch tạm thời. Không thêm thông tin xác thực hoặc kết xuất môi trường cục bộ. Giữ cho các hiện vật phiên bản hiện có được căn chỉnh.
3. Chạy trình xác thực điểm mốc hiện có và trình kiểm tra mới:

   ```bash
   node scripts/release-notes.mjs check \
     --manifest docs/releases/VERSION.contributions.json
   node scripts/release-notes.mjs render \
     --manifest docs/releases/VERSION.contributions.json \
     --output /tmp/rea-VERSION.notes.md
   ```

   Trình kiểm tra hỗ trợ nội dung PR nội tuyến, bao gồm cả metadata xung quanh của chúng, lên đến giới hạn 65.536 ký tự của GitHub. Nó báo lỗi trên các ghi chú lớn hơn thay vì cắt bớt ghi nhận; nội dung liên kết tràn của Release Please không được tích hợp.

4. Cập nhật nội dung PR phát hành với phần đã render trong khi vẫn bảo toàn header/footer và cấu trúc thành phần/phiên bản của Release Please. Sử dụng tệp nội dung cho `gh pr edit --body-file`; chỉ thay đổi `CHANGELOG.md` sẽ không thay đổi nội dung phát hành của Release Please. Chỉ push các thay đổi được ủy quyền.
5. Tìm nạp mới nội dung PR vào một tệp và chạy `check --pr-body FILE`. Xem lại toàn bộ diff và chờ CI hiện tại. Quá trình phát hành sẽ kiểm tra lại nội dung trước khi tạo bản phát hành. Tránh chỉnh sửa nội dung thêm trong khi đang phát hành.

Nhãn hoàn thiện đóng băng việc làm mới của bot, không phải nhánh main. Các thay đổi sản phẩm mới sau mục tiêu kiểm kê yêu cầu làm mới các ghi chú trước khi phát hành. Quy trình làm việc kiểm tra phả hệ mục tiêu và chỉ cho phép các đường dẫn metadata phát hành sau mục tiêu đó. Nó cũng yêu cầu mục tiêu kiểm kê phải bằng commit cha đầu tiên của merge đã xem xét, do đó các commit main xen vào yêu cầu một bản kiểm kê mới. Xem xét các diff trong các đường dẫn metadata phát hành được phép: bảo vệ đường dẫn không chứng minh rằng các chỉnh sửa phụ thuộc gói/script chỉ là thay đổi phiên bản. Để tiếp tục các đề xuất tự động, hãy xóa nhãn hoàn thiện và để bot tạo lại; các thay đổi biên tập sẽ bị thay thế.

Các bản phát hành hiện có không có bản kiểm kê đóng góp sẽ giữ nguyên đường dẫn phát hành hiện tại của chúng. Tuyệt đối không bao giờ tạo lại tag hoặc phát hành lại một gói chỉ để sửa văn bản phát hành.
