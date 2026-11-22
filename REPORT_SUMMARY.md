# Kết quả nghiên cứu

Từ báo cáo tài chính của 8.971 doanh nghiệp Mỹ, nghiên cứu xếp hạng rủi ro phá sản và ước tính chi phí lập danh sách rà soát. Mô hình chính theo quy tắc backtest 2012–2018 là **Logistic**. Biểu đồ và bảng dưới mô tả tập phát triển 2015–2018, gồm 119 sự kiện trên 12.282 quan sát (0,97%).

1. **Thứ hạng thay đổi theo năm.** XGBoost gốc có AP cao hơn Logistic trong 6/7 năm. Chênh trung bình là 0,032, p kiểm định dấu hai phía bằng 0,125. XGBoost đơn điệu đạt AP trung bình 0,226, so với 0,213 của bản gốc, trên cùng bảy năm và cùng ngân sách tìm tham số.
2. **Xác suất thay đổi khi cập nhật mức nền.** Giữ độ dốc Platt và chỉnh hệ số chặn trên 2012–2014 đưa tổng sự kiện dự báo của XGBoost từ 182,8 xuống 119,5 so với 119 quan sát. Brier skill chuyển từ 9,0% sang 8,3%.
3. **Rà soát có điểm hòa vốn.** Ở kịch bản chính, Logistic dùng ngưỡng 1,37%, đưa 1742 hồ sơ vào danh sách và tìm được 97 ca. Với chi phí 1 triệu đồng/hồ sơ, hiệu quả tránh tổn thất phải đạt ít nhất 3,99% để hòa vốn trên danh sách này.

## Mô hình gọn và ràng buộc đơn điệu

Logistic gọn giữ năm tỷ số Altman (gồm vốn hóa/nợ) và xét thêm bốn biến theo kiến thức tài chính. VIF được tính bằng train. Mô hình giữ 9 biến, AP phát triển 0,036. Chênh AP với Logistic đầy đủ -0,156 (bootstrap công ty 95% từ -0,227 đến -0,102). Đây là đánh đổi của bộ biến cụ thể. Phần so sánh XGBoost thường và đơn điệu dùng bảy backtest cùng cửa sổ và lưới.

## Chất lượng xác suất

| Năm | Ca thực tế | XGBoost OOF | XGBoost chỉnh chặn |
|---|---|---|---|
| 2015 | 33 | 54,4 | 35,6 |
| 2016 | 29 | 44,7 | 29,2 |
| 2017 | 21 | 40,9 | 26,7 |
| 2018 | 36 | 42,8 | 28,0 |

Đối với champion Logistic, Brier sau chỉnh là 0,00867, so với 0,00960 khi dùng tỷ lệ nền valid cho mọi hồ sơ. Brier skill tương ứng 9,7%. Notebook báo cả bảng theo năm lẫn decile cho hai cách hiệu chuẩn.

## Kinh tế của danh sách rà soát

| Danh sách | Rà soát | Ca đúng | e hòa vốn | C tối đa/hồ sơ¹ |
|---|---|---|---|---|
| Ngưỡng valid e=2% | 35 | 16 | 0,49% | 4,11 triệu |
| Ngưỡng valid e=5% | 847 | 78 | 2,41% | 2,07 triệu |
| Ngưỡng valid e=10% | 1742 | 97 | 3,99% | 2,51 triệu |
| Ngưỡng valid e=20% | 2184 | 100 | 4,85% | 4,12 triệu |
| Top 5% mỗi năm | 612 | 63 | 2,16% | 4,63 triệu |
| Top 10% mỗi năm | 1227 | 87 | 3,13% | 3,19 triệu |

¹ Với danh sách cố định, chi phí rà soát tối đa mỗi hồ sơ bằng e × LGD × EAD × TP / số hồ sơ. Hai dòng top dùng ngân sách theo từng năm và e = 10%. LGD = 45%, EAD = 1 tỷ đồng. Các đại lượng hòa vốn dùng kết cục phát triển đã biết.

## Kết luận

Trên dữ liệu doanh nghiệp Mỹ giai đoạn 2015–2018, mô hình Logistic giúp thu hẹp danh sách cần rà soát. Nếu mỗi năm chỉ xem 10% hồ sơ có điểm rủi ro cao nhất, danh sách gồm 1.227 hồ sơ và chứa 87 trong 119 ca phá sản (73%). Có 32 ca nằm ngoài danh sách.

Kết quả này ủng hộ việc dùng điểm mô hình để ưu tiên hồ sơ. Xác suất dự báo vẫn có sai lệch ở nhóm rủi ro cao nhất trên biểu đồ hiệu chuẩn. Logistic dự báo trung bình khoảng 4,8%, trong khi tỷ lệ phá sản thực tế của nhóm là 6,1%.

Giai đoạn 2015–2018 đã được dùng trong quá trình phát triển mô hình. Để đánh giá khả năng triển khai, cần kiểm tra trên các năm mới và đo chi phí rà soát cùng tổn thất thực sự tránh được. Điểm hòa vốn trong nghiên cứu phụ thuộc vào giả định về các đại lượng này.

Phụ lục Việt Nam dừng ở dữ liệu và nhãn nhìn trước hai năm. Trong mẫu cục bộ ban đầu, train có một ca dương. Bản phát hành chỉ chứa mã chuẩn bị dữ liệu.
