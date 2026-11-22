# Phân tích rủi ro phá sản doanh nghiệp

Notebook nghiên cứu khả năng xếp hạng rủi ro từ báo cáo tài chính và chuyển điểm thành danh sách rà soát theo chi phí giả định. Toàn bộ hàm và dữ liệu minh họa Việt Nam nằm trong `.ipynb`; không cần module Python hay JSON đi kèm. Đây là đánh giá hồi cứu; không phải hệ thống quyết định cấp tín dụng hay công cụ khuyến nghị đầu tư đã được kiểm chứng.

## Cấu trúc

| Tệp | Nội dung |
|---|---|
| `Du_bao_pha_san_doanh_nghiep.ipynb` | Phân tích, biểu đồ, diễn giải và kết quả lần chạy |
| Ô “0.1. Các hàm dùng chung” trong notebook | Đặc trưng, tiền xử lý, backtest, bootstrap, chi phí rà soát, giám sát, nhãn Việt Nam nhìn trước |
| `tests/test_research_checks.py` | Kiểm tra cửa sổ nhãn, tiền xử lý, OOF theo thời gian và ngưỡng cảnh báo |
| `requirements.txt` | Phiên bản thư viện của môi trường kiểm thử |
| `REPORT_SUMMARY.md` | Tóm tắt mục đích, dữ liệu, kết quả và giới hạn |
| `results/` | Bảng CSV và thông tin lần chạy; được cập nhật khi chạy notebook |
| Ô `VN_EMBEDDED_REPORTS` trong notebook | 105 bản ghi BCTC Việt Nam, nguồn VCI; dùng mặc định, không cần JSON hay API |
| `vn_bctc_cache.json` (tùy chọn) | Bản dự phòng/cập nhật bên ngoài; chỉ đọc khi bật `VN_USE_LOCAL_FILES`, bị loại khỏi Git |

## Dữ liệu và định nghĩa nhãn

**Mỹ:** [American Companies Bankruptcy Prediction Dataset](https://www.kaggle.com/datasets/utkarshx27/american-companies-bankruptcy-prediction-dataset), 78.682 doanh nghiệp–năm, 8.971 doanh nghiệp, năm 1999–2018. Notebook dùng `kagglehub.dataset_download` rồi đọc `american_bankruptcy.csv` trong thư mục do thư viện trả về. Không cần sao chép CSV vào project. KaggleHub dùng cache nếu phiên bản đó đã có. Tuân theo giấy phép và điều kiện tại trang nguồn khi sử dụng/chia sẻ dữ liệu.

`status_label` là nhãn theo công ty, không có ngày phá sản từng sự kiện. Quy ước chính coi năm quan sát cuối của công ty failed là năm ngay trước sự kiện; 609 nhãn dương. Không khẳng định đây là ngày phá sản đã được kiểm chứng. `results/run_manifest.json` lưu SHA-256 của CSV và phiên bản thư viện, không lưu khóa API hay đường dẫn cá nhân. Tải “mới nhất” có thể thay đổi dữ liệu; cần đối chiếu hash để tái lập đúng lần chạy.

**Việt Nam:** bản dữ liệu nhúng sao chép nguyên cache từ nguồn VCI qua Vnstock trong lần thu thập trước, 105 doanh nghiệp–năm thuộc 16 mã ngành thép, 2018–2024. Thiếu 2016–2017 và một số năm của một số doanh nghiệp. Không có dữ liệu sự kiện cảnh báo/kiểm soát/hủy niêm yết xác minh theo ngày. Phụ lục chỉ minh họa, không xếp hạng mô hình hoặc đối chiếu chất lượng với Mỹ. Nhãn dựa trên lỗ trong cả t+1 và t+2 hoặc vốn chủ âm ở một trong hai năm; X chỉ dùng năm t. Những cửa sổ thiếu tương lai bị loại, không gán thành không kiệt quệ. Bản nhúng không tự cập nhật và được chia sẻ cùng notebook; ghi rõ nguồn và tuân thủ điều kiện sử dụng dữ liệu khi phân phối.

## Môi trường và cách chạy

Môi trường đã kiểm thử: **Python 3.11.9**, seed **42**. Cài thư viện trong môi trường riêng và chọn đúng kernel đó trong Jupyter hoặc VS Code:

```bash
python -m pip install -r requirements.txt
python -m ipykernel install --user --name bankruptcy-research --display-name "Python (Bankruptcy Research)"
```

Mở notebook trong một thư mục có quyền ghi, chọn kernel vừa đăng ký, rồi **Restart Kernel → Run All**. Có thể chỉ sao chép một tệp `.ipynb` sang thư mục mới để chạy; các hàm và dữ liệu Việt Nam đã nằm bên trong. `results/` được tạo dưới working directory. Khởi động lại kernel sau khi thay phiên bản thư viện, đặc biệt NumPy/Numba/SHAP.

Phần Mỹ cần mạng để tải dữ liệu nếu chưa có cache. Phụ lục Việt Nam dùng bản dữ liệu nhúng với `VN_USE_LOCAL_FILES = False`, `VN_REFRESH = False`; không cần JSON, Vnstock hay khóa API. Không giả lập dữ liệu để lấp khoảng thiếu. Backtest và bootstrap refit làm tăng thời gian so với chỉ đánh giá một lần chia; tốc độ phụ thuộc CPU và số luồng. Seed cố định không bảo đảm từng bit giống nhau giữa mọi phiên bản/hệ điều hành.

Chạy các kiểm tra logic:

```bash
python -m unittest discover -s tests -v
```

## Thiết kế đánh giá

- Mô hình chính: train 1999–2011, valid 2012–2014, test 2015–2018. Chọn tham số và mô hình bằng PR-AUC valid. Test không dùng để chọn mô hình thay thế.
- Hiệu chuẩn Platt từ dự báo ngoài mẫu trong train: ba khối 2005–2006, 2007–2008, 2009–2011, mỗi khối có một năm đệm so với train. Fit lại toàn bộ tiền xử lý trong từng khối. Valid vẫn dùng chọn tham số và ngưỡng, nên còn thiên lệch lựa chọn.
- Baseline gồm Z cố định và Logistic trên đúng năm biến Altman. Z gốc có phạm vi ngành hẹp hơn dataset; không coi so sánh này là chứng minh ML vượt mọi phương pháp truyền thống.
- Bootstrap công ty 500 lượt cho chênh lệch mô hình và refit; dùng chung mẫu ở mỗi lượt. Không suy ra ý nghĩa thống kê bằng cách so điểm mới với khoảng tin cậy cũ.
- Backtest mở rộng 2012–2018 có inner selection chỉ dùng quá khứ, RF 100 cây/lưới nhỏ để kiểm soát thời gian. Không dùng lại tham số đã chọn trên valid tương lai. Stress test giữ train đến 2006, test 2007–2010 theo yêu cầu hồi cứu; do thiếu ngày sự kiện/công bố, không tuyên bố đây là mô phỏng đầu năm 2007 có đủ thông tin.
- Độ nhạy nhãn hai năm dùng cửa sổ quan sát chung: train đến 2010, test 2015–2016; loại cửa sổ thiếu kết cục. Không giữ riêng các ca dương ở đuôi 2017–2018 khi chưa theo dõi đủ hai năm cho nhóm âm. Loại alive rời mẫu sớm là kiểm tra hồi cứu có thể gây thiên lệch sống sót, không phải bộ lọc vận hành.
- Cờ thiếu, Logistic gọn có VIF và XGBoost đơn điệu là thí nghiệm độ vững; không chọn winner theo test. Các bảng PSI/CSI và rank ordering dùng biên từ train.

## Hành động và chi phí

Cảnh báo dẫn đến **rà soát hồ sơ**, không tự động từ chối khách hàng. Chi phí tính cho mọi cảnh báo:

```text
L = LGD × EAD
Chi phí = C_rà_soát × (TP + FP) + L × FN + (1 − e) × L × TP
Ngưỡng PD lý thuyết = C_rà_soát / (e × LGD × EAD)
```

Kịch bản chính: C = 1 triệu VND, EAD = 1 tỷ VND, LGD = 45%, hiệu quả tránh tổn thất e = 10%. Đây là giả định minh họa quy đổi, không phải dữ liệu khoản vay thực tế hay ước lượng hiệu quả can thiệp. Ngưỡng thực nghiệm chọn trên valid rồi khóa cho test. Không so sánh trực tiếp ngưỡng mới với ngưỡng 4,43% của mô hình FN:FP cũ vì bài toán chi phí và cách hiệu chuẩn đã đổi.

## Dữ liệu Việt Nam và khóa API

Đọc dữ liệu nhúng không cần Vnstock hay khóa. Bật `VN_USE_LOCAL_FILES = True` nếu muốn đọc cache/tệp bổ sung bên ngoài. Nếu muốn tải lại, bật `VN_REFRESH = True`; cần Vnstock thế hệ 6, `vnstock-oapi` và nguồn do người dùng chọn có quyền đọc BCTC; đường tải trực tuyến này chưa được kiểm thử với tài khoản nguồn của người dùng. Cài đặt theo [tài liệu Vnstock](https://vnstocks.com/docs/vnstock/v6/cai-dat), trong đúng môi trường notebook, sau khi xác nhận cấu hình phù hợp. Dữ liệu tải mới được lưu vào cache riêng, không tự thay bản nhúng trong notebook.

Nhập khóa bằng `vnstock login` và `vnstock keys set <nguồn>` trong terminal có nhập ẩn. `VN_BCTC_SOURCE` chỉ chứa tên nguồn, không phải API key. Không đặt khóa trong notebook, cell output, README hoặc Git. `.gitignore` loại `.env`, các kho khóa cục bộ và cache Việt Nam. Gitignore không xóa bí mật đã commit; nếu từng lộ, phải thu hồi/đổi khóa và xử lý lịch sử Git.

## Giới hạn còn lại

Nhãn sự kiện suy ra, test ít ca và đã được xem trong quá trình nghiên cứu. Bootstrap công ty không mô tả đầy đủ phụ thuộc vĩ mô hoặc bất định từ fit/chọn tham số. OOF được tạo với tham số đã chọn theo valid nên vẫn không phải một nghiên cứu xác nhận độc lập hoàn toàn. Không có ngày công bố thực tế, ngành, LGD/EAD theo hồ sơ hoặc dữ liệu can thiệp. Kết quả kiểm tra thêm không khắc phục được thiếu dữ liệu nguồn. Cần tập ngoài thời gian mới và hồ sơ sự kiện xác minh trước triển khai.
