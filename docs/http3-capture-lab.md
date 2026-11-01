# Xác minh bản thu thập HTTP/3 MITM thực tế

Phòng thí nghiệm Linux tùy chọn này kiểm tra các bản thu thập mitmproxy native thông qua CLI và MCP công khai của REA. Nó không làm thay đổi các hợp đồng hay adapter runtime và chỉ xác lập trình tạo, chế độ và các client cụ thể được thử nghiệm tại đây.

## Chuẩn bị máy khách (Guest) dùng một lần

Hình ảnh được thử nghiệm là [Omarchy 4.0.4, build 440](https://nightly.omarchy.org/stable/4.0.4/440/omarchy-4.0.4.qcow2), SHA-256 `43412b896fe3d730e373a51a8acf99343d7c3e6dee78aa9f9bd7fca181127264`. Tải xuống mà không cài đặt các gói guest trên máy chủ host, kiểm tra mã băm của nó và sử dụng một **qcow2 overlay mới** với tệp đó làm hình ảnh nền chỉ đọc.

Máy khách kín này đã cung cấp sẵn cURL 8.22.0 với OpenSSL 3.6.4, ngtcp2 1.25.0, nghttp3 1.18.0 và nghttp2 1.70.0. **Không cần nâng cấp gói cuốn chiếu (rolling) của máy khách.** Chạy `curl --version` và xác minh các tính năng HTTP2 và HTTP3 trước khi bắt đầu.

Ghi lại mã băm hình ảnh, bản sửa đổi nguồn của REA và các phiên bản phụ thuộc này:
- `.nvmrc` của REA, `package.json#packageManager` và các phụ thuộc npm đã khóa.
- Kho lưu trữ độc lập mitmproxy Linux x86_64 chính thức **12.2.3**. SHA-256: `2e95286b618fa6fd33e5e62a78c2e5112571d85f42ec2bac29b97ee242bdb5c5`.
- Một môi trường ảo Python riêng tư sử dụng đồ thị ghim đầy đủ trong `scripts/fixtures/http3-lab-requirements.txt`.
- OpenSSL và cURL quảng bá HTTP3 và HTTP2 trong `curl --version`.

Từ bản checkout REA của máy khách, hãy sử dụng chuỗi công cụ Node/npm đã ghim của nó:

```sh
npm ci
npm run build:cached
python3 -m venv /absolute/lab/venv
/absolute/lab/venv/bin/pip install --no-deps -r scripts/fixtures/http3-lab-requirements.txt
/absolute/lab/venv/bin/pip check
/absolute/lab/venv/bin/pip freeze > /absolute/lab/python-requirements.txt
```

Trước khi xóa định tuyến mặc định của máy khách, hãy tải xuống kho lưu trữ mitmproxy chính thức và xác minh nó:

```sh
mkdir -p /absolute/lab/tools
curl --fail --location https://downloads.mitmproxy.org/12.2.3/mitmproxy-12.2.3-linux-x86_64.tar.gz   --output /absolute/lab/mitmproxy.tar.gz
printf '%s  %s
' 2e95286b618fa6fd33e5e62a78c2e5112571d85f42ec2bac29b97ee242bdb5c5   /absolute/lab/mitmproxy.tar.gz | sha256sum --check --strict
tar -xzf /absolute/lab/mitmproxy.tar.gz -C /absolute/lab/tools
/absolute/lab/tools/mitmdump --version
```

## Chạy profile hẹp

Chọn các tệp thực thi tuyệt đối và một thư mục bằng chứng mới:

```sh
REA_MITMDUMP_COMMAND=/absolute/lab/tools/mitmdump REA_HTTP3_PYTHON_COMMAND=/absolute/lab/venv/bin/python npm run verify:web:http3 -- /absolute/evidence/run-1
```

Trình tạo gửi ba yêu cầu POST nhị phân đồng thời trên một kết nối HTTP/3 đến cùng một URL thông qua chế độ đảo ngược (reverse mode) của mitmproxy tới một origin HTTP/3 aioquic. Addon lưu thực tế của mitmproxy ghi `normal.mitm`; không có luồng native nào được tổng hợp.
