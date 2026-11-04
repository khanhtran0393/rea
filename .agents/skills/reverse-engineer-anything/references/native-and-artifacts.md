# Hiện vật Native, Managed và Đóng gói (Native, managed, and packaged artifacts)

Đối với các so sánh bản phát hành (release comparisons), hãy xuất Evidence từ `list_procedures` của từng mục tiêu trước khi đóng nó. Thao tác đóng sẽ xóa sạch các bản ghi phiên: hãy import cả hai gói bundle sau khi thu thập hai mục tiêu, hoặc cung cấp Evidence nội tuyến khả chuyển. Lệnh `match_native_functions` ghép cặp các tên duy nhất chính xác và các byte thân hàm hoàn chỉnh quan sát được (nếu được cung cấp). Tái sử dụng `list_names`, `procedure_info`, `read_bytes` tùy chọn cùng ngữ cảnh và các hồ sơ `analyze_function` hiện có dưới dạng `left_observations`/`right_observations`; câu trả lời tóm tắt của `analyze_function` qua MCP đóng góp `parent_evidence_id` của nó làm tham chiếu `retained-evidence`, không phải bản thân bản tóm tắt. Tránh dịch ngược mọi hàm chỉ để so sánh khớp nó. Bản tóm tắt của nó lưu giữ một ID cha cho `inspect_analysis_view` các tập hợp `function-matches` gồm `matched_pairs`, `unmatched_left` và `unmatched_right`.

## Tài nguyên ứng dụng Apple (Apple app resources)

Sử dụng `open_bundle_resources` với đường dẫn ứng dụng đã chọn, sau đó là `inspect_plist`, `decode_interface_builder`, `inspect_asset_catalog` hoặc `inspect_keyed_archive`. Phiên tài nguyên sẽ đọc ứng dụng mà không cần phân giải `CFBundleExecutable` hoặc khởi động một provider chuyên sâu. Bảo toàn danh tính phiên `resource_session`, SHA-256 của tài nguyên và các phân cấp đã giải mã. Không có bản chụp nhị phân hoặc mã băm tệp thực thi nào được xác lập; sử dụng `open_binary` cho mã native.

## Thư mục cài đặt ứng dụng Desktop (Desktop installation directories)

Khi một thư mục cài đặt được cung cấp và runtime của nó chưa xác định, hãy sử dụng `inspect_desktop_application` với đường dẫn tuyệt đối của nó. Phản hồi tóm tắt mặc định của MCP sẽ lưu giữ Evidence đầy đủ. Đọc `files`, `omissions`, `candidates`, `components` hoặc `next_probes` bằng `inspect_analysis_view`. Danh mục kiểm kê phát hiện các dấu hiệu runtime phổ biến (Electron, .NET, Java, Python, Go, Rust, Qt, Native Windows) và đề xuất bước tiếp theo phù hợp.

## Metadata bản dựng Go (Go build metadata)

Sử dụng `inspect_go_binary` cho tệp ELF, PE hoặc Mach-O cục bộ rõ ràng (tệp universal cần chỉ định `architecture`) khi câu hỏi liên quan đến phiên bản trình biên dịch được nhúng, phiên bản mô-đun/phụ thuộc, các thay thế cục bộ hoặc thiết lập bản dựng. Không yêu cầu `open_binary`, không cần cài đặt Go hoặc thực thi mục tiêu. CLI tương đương là `inspect-go-binary PATH`. Bảo toàn các byte mô-đun thô và phạm vi mã nguồn.

## Tệp mô hình (Model files)

Sử dụng `inspect_model_artifact` (`inspect-model-artifact PATH`) cho các khai báo tệp GGUF v2/v3 và safetensors cục bộ mà không cần runtime mô hình. Chế độ chỉ đọc header không đọc trọng số và để lại danh tính toàn bộ tệp là chưa xác định. Tùy chọn `hash: "full"` đọc luồng tất cả các byte cho SHA-256. Các chế độ xem `model-artifact` được lưu giữ phân trang các khóa metadata, tensor và hình dạng.

## Metadata IL2CPP (IL2CPP metadata)

Sử dụng `inspect_il2cpp_metadata` cho tệp Unity `global-metadata.dat` cục bộ rõ ràng (hỗ trợ các phiên bản 24, 27, 29, 31 và 39) khi câu hỏi liên quan đến ảnh assembly, tên, đối tượng sở hữu thành viên, chuỗi ký tự, không gian tên hoặc phạm vi chỉ số. Không yêu cầu `open_binary`, cài đặt Unity hoặc thực thi mục tiêu. CLI tương đương là `inspect-il2cpp-metadata PATH`.

## Metadata phụ thuộc Rust (Rust dependency metadata)

Sử dụng `inspect_rust_binary` cho tệp ELF, PE hoặc Mach-O cục bộ rõ ràng (tệp universal cần `architecture`) khi câu hỏi liên quan đến danh sách crate cargo-auditable được nhúng trong `.dep-v0` (Mach-O: `__DATA,.dep-v0`). Không yêu cầu `open_binary`, cài đặt Rust hoặc thực thi mục tiêu. CLI tương đương là `inspect-rust-binary PATH`.

## Bảng chứng chỉ Authenticode PE (PE Authenticode certificate tables)

Sử dụng `inspect_pe_certificates` cho tệp ảnh PE32/PE32+ cục bộ rõ ràng khi câu hỏi liên quan đến bảng chứng chỉ Authenticode, các kiểu WIN_CERTIFICATE, hoặc các trường danh tính người ký PKCS#7 và X.509 được nhúng. Không yêu cầu `open_binary`, Windows SDK hoặc thực thi mục tiêu. CLI tương đương là `inspect-pe-certificates PATH`. Chữ ký không được tự động xác minh độ tin cậy.

## Bảng import và delay-load PE (PE import and delay-load tables)

Sử dụng `inspect_pe_imports` cho tệp ảnh PE32/PE32+ cục bộ rõ ràng khi câu hỏi liên quan đến tên DLL và hàm được import, số thứ tự (ordinals), bộ mô tả delay-load, hoặc mã băm import pefile. Không yêu cầu `open_binary`, Windows SDK hoặc thực thi mục tiêu. CLI tương đương là `inspect-pe-imports PATH`.

## TLS callbacks và cấu hình nạp PE (PE TLS callbacks and load configuration)

Sử dụng `inspect_pe_load_config` cho tệp ảnh PE32/PE32+ cục bộ rõ ràng khi câu hỏi liên quan đến mã chạy trước điểm vào thông qua TLS callbacks, hoặc các khai báo security cookie, SafeSEH, CFG, EH-continuation và XFG trong cấu hình nạp (load configuration). Không yêu cầu `open_binary`, Windows SDK hoặc thực thi mục tiêu. CLI tương đương là `inspect-pe-load-config PATH`.

## Bảng ELF note (ELF note tables)

Sử dụng `inspect_elf_notes` cho tệp ảnh ELF32/ELF64 cục bộ rõ ràng khi câu hỏi liên quan đến chủ sở hữu PT_NOTE/SHT_NOTE, GNU build-id, hoặc các trường GNU ABI-tag. Không yêu cầu `open_binary` hoặc thực thi mục tiêu. CLI tương đương là `inspect-elf-notes PATH`. Bảo toàn SHA-256 của hiện vật, các phạm vi bảng và phần mô tả.

## Báo cáo sự cố Apple IPS (Apple IPS crash reports)

Sử dụng `inspect_apple_ips` cho tệp Apple `.ips` cục bộ rõ ràng khi câu hỏi liên quan đến metadata sự cố, dữ kiện ngoại lệ và kết thúc, các luồng, usedImages, thanh ghi số nguyên không mất dữ liệu, hoặc các lớp sự cố đã được tài liệu hóa. Không yêu cầu `open_binary`, Xcode, atos hoặc thực thi mục tiêu. CLI tương đương là `inspect-apple-ips PATH`.

## Các mô-đun kernel Linux (Linux kernel modules)

Sử dụng `inspect_kernel_module` cho tệp đối tượng tái định vị (relocatable) ELF32/ELF64 cục bộ rõ ràng khi câu hỏi liên quan đến license trong `.modinfo`, vermagic, alias, depends, hoặc chuỗi parm, CRC ký hiệu `__versions`, hoặc phần đuôi chữ ký mô-đun được thêm vào. Không yêu cầu `open_binary`, máy chủ Linux hoặc lệnh `modinfo`. CLI tương đương là `inspect-kernel-module PATH`. Các gói nén gzip, xz và zstd không được giải nén trực tiếp; hãy giải nén ra tệp riêng trước khi kiểm tra.

## Tệp pyc CPython (CPython pyc files)

Sử dụng `inspect_python_pyc` cho tệp CPython 3.7–3.13 `.pyc` cục bộ rõ ràng khi câu hỏi liên quan đến header magic, mã băm nguồn hoặc dấu thời gian PEP 552, tên đối tượng mã lồng nhau, hoặc danh tính bytecode. Không yêu cầu `open_binary`, cài đặt Python hoặc thực thi mục tiêu. CLI tương đương là `inspect-python-pyc PATH`.

## Tệp Erlang BEAM (Erlang BEAM files)

Sử dụng `inspect_erlang_beam` cho tệp `FOR1`/`BEAM` cục bộ rõ ràng khi câu hỏi liên quan đến chunk IFF, tên atom, exports, imports, lambdas hoặc header Code. Không yêu cầu `open_binary`, cài đặt Erlang hoặc thực thi bytecode. CLI tương đương là `inspect-erlang-beam PATH`.

## Ảnh chụp cơ sở dữ liệu SQLite (SQLite snapshots)

Sử dụng `inspect_sqlite_database` với ảnh chụp cơ sở dữ liệu cục bộ rõ ràng để lấy schema của nó và các hàng tùy chọn từ một bảng thông thường chính xác. Yêu cầu Node.js 24.x >=24.15 hoặc 26+ với API native `DatabaseSync.limits`. Nó cũng đọc tệp `-wal` liền kề hiện có; lưu giữ nguồn gốc của cả hai tệp.

## Sự cố Linux đã ghi nhận (Recorded Linux crashes)

Sử dụng `inspect_recorded_crash` với đường dẫn tệp core Linux x86-64 ELF rõ ràng để kiểm tra các thanh ghi luồng lịch sử, tín hiệu, ghi chú thô và phạm vi tệp gốc. Hoạt động độc lập với mục tiêu của bộ tháo mã đang hoạt động. Các giá trị thanh ghi ràng buộc nguồn và byte ghi chú là các quan sát thực tế; ánh xạ bị thiếu vẫn là chưa xác định.

## Bố cục ELF ngoại tuyến (Offline ELF layout)

Sử dụng `inspect_binary_layout` với đường dẫn ELF cục bộ rõ ràng để kiểm tra các section được tệp hỗ trợ, ký hiệu, tái định vị, tên liên kết và suy luận giảm thiểu rủi ro tĩnh mà không cần mở mục tiêu bộ tháo mã. Khi bản ghi đầy đủ đó đã được lưu giữ, hãy gọi `inspect_analysis_view` để lấy bản tóm tắt hoặc kiểm tra từng section/ký hiệu.

## WASM ngoại tuyến (Offline WASM)

Sử dụng `inspect_wasm_artifact` với WABT do bên gọi cung cấp cho các byte cục bộ đã chọn. Đối với số lượng, xác thực và danh tính WAT, hãy chọn `detail: "summary"` trong lệnh gọi MCP đầu tiên. Evidence ban đầu đầy đủ vẫn được lưu giữ; sử dụng `normalized_result.parent_evidence_id` cho các export chính xác hoặc các lát cắt văn bản.

## Chữ ký APK ngoại tuyến (Offline APK signatures)

Sử dụng `inspect_apk_signature` cho tệp APK/ZIP/AAB cục bộ đã chọn. Bản tóm tắt MCP mặc định lưu giữ Evidence đầy đủ. Chọn `view.collection: signers/digests/lineage`, với các lô scheme hoặc ID, các trường và phân trang; sử dụng `inspect_analysis_view` loại `apk-signature` cho các lần đọc được lưu giữ. Báo cáo các cơ chế chữ ký hiện diện (v1, v2, v3, v3.1), dấu vân tay chứng chỉ SHA-256 của từng người ký và tính nhất quán nội tại của các mã băm.

## Các mục tiêu Native (Native targets)

Sau khi gọi `open_binary`, hãy sử dụng trực tiếp các công cụ tìm kiếm, thủ tục hoặc hàm tập trung. Sử dụng `binary_overview` khi metadata mục tiêu hoặc ngữ cảnh kiểm kê giúp trả lời câu hỏi. Ưu tiên tìm kiếm ký tự, tên, dịch ngược (decompilation), callers, callees và cross-references. Địa chỉ và mã giả được khôi phục là dữ kiện phân tích tĩnh; chúng không quan sát hoặc chứng minh quá trình thực thi runtime.

Nếu có cấu hình IDA MCP, hãy sử dụng `open_binary` với `provider_id: "ida"`. Khi cấu hình `REA_KUNA_COMMAND`, `provider_id: "kuna"` cung cấp danh sách thủ tục, mã giả và hợp ngữ từ bản xuất toàn bộ chương trình được lưu trong bộ nhớ đệm.

## Tái sử dụng phân tích Ghidra do người gọi chọn (Reuse caller-selected Ghidra analysis)

Khi bên gọi cung cấp một dự án đã mở rộng, hãy cấu hình `REA_GHIDRA_PROJECT_PATH` (dấu hiệu `.gpr`) và tùy chọn `REA_GHIDRA_PROGRAM_NAME` (một chương trình gốc), sau đó khởi động lại/kết nối lại trước khi gọi `open_binary`. CLI sử dụng cùng một cấu hình đó. REA chỉ mở một tập hợp chương trình hữu hạn; các chương trình khác trong dự án vẫn chưa được quan sát.

## Khôi phục thời hạn khởi động Ghidra (Ghidra startup deadline recovery) <a id="ghidra-startup-deadline-recovery"></a>

Lỗi `provider_timeout` trong truy vấn Ghidra đầu tiên có thể bao gồm việc import, tự động phân tích (auto-analysis), khởi động bridge và kiểm tra tính sẵn sàng. Mức mặc định 330,000 ms có thể cấu hình thông qua `REA_GHIDRA_STARTUP_TIMEOUT_MS`; một truy vấn nhỏ hơn vẫn cần cùng một lần import khi cold.

Khi lỗi chỉ ra một thời hạn khởi động đã hết hạn, hãy tăng giá trị thời hạn thông qua `update_analysis_settings` với `ghidra.startup_timeout_ms` trước khi thử lại, hoặc khởi động lại server với biến môi trường được cập nhật. Đảm bảo thời gian chờ của MCP client đủ lớn để chấp nhận thời hạn đó.

## Giới hạn trần yêu cầu Bridge (Bridge request ceiling)

Hopper và Cutter chia sẻ `REA_BRIDGE_MAX_REQUEST_BYTES`. Nếu chưa đặt, cả hai bên sử dụng 16,777,216 byte. Ngân sách payload không bao gồm ký tự kết thúc dòng LF. Một số nguyên thập phân từ 7464 đến 536870887 sẽ nâng hoặc hạ mức trần đó.

## Dữ kiện native và đồ thị cuộc gọi đã chọn (Selected native facts and call graphs)

Đối với một `get_call_graph` lớn, hoặc các truy vấn lặp lại `procedure_address`, `procedure_callees`, `procedure_callers` và `resolve_containing_procedure`, hãy chọn `detail: "summary"` khi 64 quan sát đầu tiên được ghi lại là đủ. Đọc các trang `native-observation` tiếp theo bằng `inspect_analysis_view` với `parent_evidence_id` được trả về. Lệnh `build_call_path` yêu cầu các hồ sơ hàm hoàn chỉnh từ cùng một hiện vật/provider.

## Bộ nhớ tĩnh và các byte tệp gốc (Static memory and original file bytes)

Đối với Ghidra, `read_bytes` đọc bộ nhớ phân tích đã khởi tạo, bao gồm các chỉnh sửa của trình nạp (loader fixups); nó không đọc tiến trình đang chạy. Kiểm tra `returned_bytes` và `complete` trước khi giải mã một bảng. Một lần đọc chưa hoàn chỉnh là thiếu độ bao phủ, không phải là phần đuôi chứa số 0. Sử dụng `list_segments` để xác định các ranh giới ánh xạ hợp lệ.

## DOS MZ với Ghidra (DOS MZ with Ghidra)

Đối với mục tiêu DOS MZ được chấp nhận, hãy sử dụng `inspect_native_load_image` để kiểm tra các byte đã nạp được đo lường, ánh xạ tệp, tái định vị và điểm vào dựa trên bản chụp bất biến. Giữ `mismatch` và `unsupported` rõ ràng. Việc import đã xác minh chỉ bao gồm hình ảnh tĩnh được báo cáo, không phải DOS runtime.

## Managed PE/CLI (.NET NativeAOT)

Bắt đầu với `inspect_managed_artifact`. Việc kiểm tra managed chuẩn của REA là hoàn toàn không thực thi: không tuyên bố rằng nó đã nạp, phản chiếu (reflected), thực thi hoặc phân giải assembly. Giữ ranh giới managed/native và các dữ kiện tái cấu trúc không khả dụng một cách rõ ràng.

Đối với .NET NativeAOT (PE hoặc ELF), hãy sử dụng phân tích Ghidra native. Mã native có thể tạo ra mã giả; tham khảo quy trình NativeAOT để khôi phục metadata tùy chọn và ranh giới hỗ trợ về host/layout.

## Gói và giải nén (Packages and extraction)

Sử dụng `inspect_artifact` cho các gói ứng dụng, kho lưu trữ, đầu vào ZIP/APK/IPA/MSIX/AppX, ASAR hoặc DMG khi đồ thị hiện vật và các phát hiện giúp trả lời câu hỏi. Nó trả về toàn bộ đồ thị hiện vật nội tuyến. Trích dẫn manifest ID của đồ thị khi sử dụng chúng.

Đối với tệp thực thi PyInstaller, PyInstaller CArchive chính là tầng ứng dụng; bootloader chỉ là phần tiền tố native. Bắt đầu với `detail: "summary"` cho các kho kiểm kê lớn, sau đó sử dụng các chế độ xem kiểm kê được lưu giữ. Đối với ảnh firmware, sử dụng `extract_firmware` khi được yêu cầu, với thư mục xuất tuyệt đối mới do bên gọi chọn (sử dụng Binwalk/Unblob trên Linux).

### Chú thích hàm trong Ghidra (Function annotations in Ghidra)

Khi làm việc với Ghidra, bạn có thể kiểm tra và áp dụng các chú thích hàm, kiểu dữ liệu và tên tùy chỉnh để làm rõ logic đã dịch ngược mà không làm hỏng tính toàn vẹn của tệp nhị phân gốc.

## Kiểm kê tài nguyên PE ngoại tuyến (Offline PE resource inventory)

Sử dụng `inspect_pe_resources` với một `path` tuyệt đối, hoặc `rea inspect-pe-resources ./selected.exe --json`, để kiểm tra tài nguyên PE32/PE32+ khả chuyển mà không cần mở mục tiêu native. Bảo toàn Evidence được trả về: ID kiểu/tên/ngôn ngữ và tên UTF-16, vị trí thư mục gốc và phạm vi tệp payload cùng SHA-256.

## Bảng export PE và ELF ngoại tuyến (Offline PE and ELF export tables)

Sử dụng `inspect_native_exports` với một `path` tuyệt đối, hoặc `rea inspect-native-exports ./libexample.so --json`, để đọc thư mục export PE hoặc bảng ký hiệu động ELF trên bất kỳ máy chủ nào mà không cần mở mục tiêu native. Các export chỉ theo số thứ tự (ordinal-only), được chuyển tiếp (forwarded) và phiên bản ẩn vẫn được giữ rõ ràng.

## Godot PCK

Đối với tệp PCK độc lập, gọi `open_binary` sau đó `inspect_artifact` với `detail: "summary"` sẽ lưu giữ toàn bộ danh mục kiểm kê tĩnh. Sử dụng `parent_evidence_id` được trả về với các lần xuất hiện kiểm kê của `inspect_analysis_view` được lọc theo `logical_paths` hoặc `occurrence_ids` chính xác, và nối `artifact_id` để kiểm tra các tài nguyên nhúng.

## Ảnh ESP (ESP images)

Đối với ảnh ứng dụng hoặc bootloader dòng ESP32, gọi `open_binary` sau đó `inspect_artifact` với `detail: "summary"`. Trường `root_format_metadata` chứa `header` và `descriptor` quan sát được (trường `kind` của nó là vai trò ứng dụng hoặc bootloader) cùng `chip`, `checksum`, `appended_sha256` và các section được suy ra.

## Ảnh flash ESP (ESP flash images)

Bản dump toàn bộ flash sẽ mở dưới dạng `esp-flash-image`. Trường `root_format_metadata` chứa `partition_table` (phương thức của nó: `default`, `caller` hoặc `scan` được suy ra), dữ kiện ảnh ESP của `bootloader` và `apps`, các bản ghi `otadata`, `partitions_outside_dump` và `boot_selection`.

## Tệp thực thi độc lập Bun (Bun standalone executables)

Tệp thực thi `bun build --compile` lưu trữ đồ thị mô-đun của nó trong Mach-O `__BUN,__bun`, một section `.bun` của PE hoặc ELF, hoặc phần đuôi được thêm vào. Lệnh `open_binary` sau đó `inspect_artifact` báo cáo định dạng gốc `bun-standalone` cùng carrier, bố cục cấu trúc, số lượng mô-đun, điểm vào và các mô-đun được lưu trữ để trích xuất và phân tích JavaScript tiếp theo.
